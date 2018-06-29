---
title: gstreamer多媒体
date: 2017-07-20 18:06:39
tags: [gstreamer]
---

# 前言

因为公司的需要，研究使用了一段时间的`gstreamer`框架。`gstreamer`是一个多平台的多媒体框架，具有较高的扩展性，由具有各种功能的元件组成。这里作简单的总结。

# 基本用法

## 元件的创建

元件通过工厂函数创建，类型为`GstElement`，元件基本上都有`name`属性，用来给元件命名，也可以利用这个名字来获取元件，这里不介绍。

```
GstElement *pipeline, *appsrc, *rtph264depay, *decoder, *conv, *videosink;

pipeline = gst_pipeline_new("pipeline_sender");
src = gst_element_factory_make("autovideosrc", "source_sender");
rate = gst_element_factory_make("videorate", "rate_sender");
convert = gst_element_factory_make("videoconvert", "convert_sender");
encoder = gst_element_factory_make("openh264enc", "encoder_sender");
rtph264pay = gst_element_factory_make("rtph264pay", "rtppay_sender");
sink = gst_element_factory_make("appsink", "appsink_sender");
```

## 设置属性

每个元件具有很多属性，经常需要对有些元件进行属性的设置。通常有两种方式设置，可以通过设置CAPS，或者直接设置对象属性。

通过CAPS设置

```
#define VIDEO_RTP_SENDER "application/x-rtp,media=(string)video,payload=(int)96,clock-rate=(int)90000,encoding-name=(string)H264"

GstCaps *caps;
caps = gst_caps_from_string(VIDEO_RTP_SENDER);
g_object_set(G_OBJECT(sink), "caps", caps, NULL);
gst_caps_unref(caps);
```

直接设置对象属性

```
g_object_set(G_OBJECT(sink), "emit-signals", TRUE, "sync", FALSE, NULL);
```

## 通道连接

只有将元件连接成通道，才能正确使用其功能，注意在连接通道之前需要先设置好属性，然后将其他元件加入到通道元件中。

```
gst_bin_add_many(GST_BIN(pipeline), src, rate, convert, encoder, rtph264pay, sink, NULL);

if (!gst_element_link_many(src, rate, convert, encoder, rtph264pay, sink, NULL))
	qDebug() << "fail to link sender pipeline";
```

## 运行和关闭

通过操作通道的属性可以控制通道的运行和关闭，如果需要该通道循环运行，可以创建一个loop循环对象。

```
GMainLoop *loop;
loop = g_main_loop_new(NULL, FALSE);
```

运行时代码

```
gst_element_set_state(data->pipeline, GST_STATE_PLAYING);
g_main_loop_run(data->loop);
```

关闭时代码

```
g_main_loop_quit(data->loop);
gst_element_set_state(data->pipeline, GST_STATE_NULL);
```

# appsrc和appsink

当数据流到`appsink`时可以通过信号调用相应的回调函数，传递数据。

```
g_signal_connect(data->sink, "new-sample",
	G_CALLBACK(on_new_sample_from_sink), data);

GstFlowReturn 
VideoSender::on_new_sample_from_sink(GstElement * elt, VideoSenderData * data)
{
	GstSample *sample;
	GstBuffer *buffer;
	GstMapInfo map;
	sample = gst_app_sink_pull_sample(GST_APP_SINK(elt));
	buffer = gst_sample_get_buffer(sample);
	gst_buffer_map(buffer, &map, GST_MAP_READ);
	gst_sample_unref(sample);

	/*
		...省略数据的处理...
	*/

	send(data->sock, (char *)(map.data), map_size, 0);

	qDebug() << "video send size: " << total_size;

	return GstFlowReturn(0);
}
```


`appsrc`和`appsink`元件允许程序员操作通道中的源数据。

`appsrc`可以采用循环`push`的方式将数据传给通道，将`block`属性设为`true`之后，`gst_app_src_push_buffer`函数可以在缓冲区满的情况下阻塞。

```
g_object_set(data->appsrc,
	"format", GST_FORMAT_TIME,
	"blok", TRUE,
	NULL);

void
VideoRec::prepare_data()
{
	guint8 size_buffer[6];
	gsize rec_size = 0;
	gsize client_rec_size;

	/*
		....此处省略数据的获取部分...
	*/

	GstBuffer *buffer;
	buffer = gst_buffer_new_wrapped((gpointer)(frame_buffer), map_size);

	gst_app_src_push_buffer(GST_APP_SRC(data->appsrc), buffer);
}

flag = 1;
while (flag)
{
	prepare_data();
}
```

# 关于h264编解码的延时

在使用x264enc作为编码元件的时候总是存在两秒的延时，尽管设置了不少参数，依然没有什么效果，最后更换成`openh264enc`和`openh264dec`之后延时又不见了，参数都不用设置，比较奇怪。

# windows环境下的安装

Windiws 安装 `gstreamer` 需要下载两个安装包，分别是devel安装包和普通安装包。

这里选择的是：`gstreamer-1.0-x86_64-1.11.2.msi`
	      `gstreamer-1.0-devel-x86_64-1.11.2.msi`

选择安装完整版，安装完成后，D盘会多了一个gstreamer目录。

添加目录`E:\gstreamer\1.0\x86_64\bin`到环境变量。

Visual Studio配置：

C/C++ -> 常规 -> 附加包含目录:
`D:\gstreamer\1.0\x86_64\include\gstreamer-1.0;`
`D:\gstreamer\1.0\x86_64\lib\gstreamer-1.0\include;`
`D:\gstreamer\1.0\x86_64\include\glib-2.0;`
`D:\gstreamer\1.0\x86_64\include\libxml2;`
`D:\gstreamer\1.0\x86_64\include;`
`D:\gstreamer\1.0\x86_64\lib\glib-2.0\include;`
`%(AdditionalIncludeDirectories)`

链接器 -> 常规 -> 附加库目录：
D:\gstreamer\1.0\x86_64\lib;
%(AdditionalLibraryDirectories)

也可以将所有的lib文件添加到资源管理

使用到socket编程，需要在链接器 -> 常规 -> 输入 里面再添加ws2_32.lib
