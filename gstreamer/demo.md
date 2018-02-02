# 播放一个amr的音频文件

头文件如下:
```
#include "gst/gst.h"
#include <thread>

typedef struct
{
	GMainLoop *loop;
	GstElement *pipeline;
	GstElement *filesrc, *audioparse, *audiodec, *audioconvert, *audiosink;
}AudioPlayerData;

class AudioPlayer
{
public:
	explicit AudioPlayer(QString path);
	~AudioPlayer();

	void start();
	void stop();
	bool is_running();

private:
	void run();

private:
	AudioPlayerData *data;

	std::thread t;
};


```

播放amr音频文件需要用到以下元件:

+ filesrc
+ amrparse
+ amrnbdec
+ audioconvert
+ autoaudiosink

因为需要监听播放完成事件，所以创建一个mainloop:
```
data->loop = g_main_loop_new(NULL, FALSE);
```

创建各个元件:
```
data->pipeline = gst_pipeline_new("audio_player");
data->filesrc = gst_element_factory_make("filesrc", "audio_player_filesrc");
data->audioparse = gst_element_factory_make("amrparse", "audio_player_parse");
data->audiodec = gst_element_factory_make("amrnbdec", "audio_player_dec");
data->audioconvert = gst_element_factory_make("audioconvert", "audio_player_convert");
data->audiosink = gst_element_factory_make("autoaudiosink", "audio_player_sink");
```

指定一下filesrc文件路径:
```
g_object_set(data->filesrc,
	"location", path.toStdString().c_str(),
	NULL);
```

从pipeline中获取一个GstBus对象，然后设置监听并绑定一个回调函数。
```
GstBus *bus = gst_pipeline_get_bus(GST_PIPELINE(data->pipeline));
gst_bus_add_watch(bus, call_back, data);
gst_object_unref(bus);
```
定义一个回调函数， 当获取到播放完成或错误事件的时候关闭通道。
```
static gboolean call_back(GstBus *, GstMessage *msg, gpointer data)
{
	switch (GST_MESSAGE_TYPE(msg))
	{
		case GST_MESSAGE_EOS:
		case GST_MESSAGE_ERROR:
			gst_element_set_state(static_cast<AudioPlayerData*>(data)->pipeline, GST_STATE_NULL);
			break;
		default:
			break;
	}

	return true;
}
```

将元件添加到pipeline并连接
```
gst_bin_add_many(GST_BIN(data->pipeline), data->filesrc, data->audioparse,
	data->audiodec, data->audioconvert, data->audiosink, NULL);

if (!gst_element_link_many(data->filesrc, data->audioparse, data->audiodec, data->audioconvert, data->audiosink, NULL))
{
	qDebug() << "fail to link audio player";
}
```

提供一些简单的接口，用于查询控制播放，这里用一个单独的线程跑loop：
```
void AudioPlayer::start()
{
	t = std::thread(&AudioPlayer::run, this);
}

void AudioPlayer::stop()
{
	gst_element_set_state(data->pipeline, GST_STATE_NULL);
	g_main_loop_quit(data->loop);
	if (t.joinable())
		t.join();
}

bool AudioPlayer::is_running()
{
	GstState state;
	gst_element_get_state(data->pipeline, &state, NULL, GstClockTime(1000000));
	return state == GST_STATE_PLAYING;
}

void AudioPlayer::run()
{
	gst_element_set_state(data->pipeline, GST_STATE_PLAYING);
	g_main_loop_run(data->loop);
}

```

析构的时候关闭通道释放资源：
```
AudioPlayer::~AudioPlayer()
{
	gst_element_set_state(data->pipeline, GST_STATE_NULL);
	g_main_loop_quit(data->loop);
	if(t.joinable())
		t.join();
	g_object_unref(data->pipeline);
	g_free(data);
}

```

# 生成一个amr音频文件

头文件如下:
```
#include "gst/gst.h"

typedef struct
{
	GstElement *pipeline;
	GstElement *audiosrc, *audioconv, *audioenc, *filesink;
}ShortAudioData;

class ShortAudio
{
public:
	explicit ShortAudio(QString path);
	~ShortAudio();

	void start();
	void stop();

private:
	ShortAudioData *data;
};

```

生成amr用到以下元件:

+ autoaudiosrc
+ audioconvert
+ amrnbenc
+ filesink

生成的amr文件是不能直接播放的，需要添加头:
```
const char header[6] = { 0x23, 0x21, 0x41, 0x4d, 0x52, 0x0a};
std::ofstream of(path.toStdString(), std::ofstream::binary);
of.write(header, 6);
```

设置码率和模式:
```
g_object_set(data->audioenc, 
	"target", "bitrate",
	"bitrate", 192,
	NULL);
```

设置文件路径，因为已经输入了头，所以指定以append的方式添加数据:
```
g_object_set(data->filesink,
	"append", true,
	"location", path.toStdString().c_str(),
	NULL);
```

连接:
```
gst_bin_add_many(GST_BIN(data->pipeline), data->audiosrc, data->audioconv,
	data->audioenc, data->filesink);

if (!gst_element_link_many(data->audiosrc, data->audioconv, data->audioenc, data->filesink, NULL))
{
	qDebug() << "fail to link short video";
}
```

提供简单的接口：
```
void ShortAudio::start()
{
	gst_element_set_state(data->pipeline, GST_STATE_PLAYING);
}

void ShortAudio::stop()
{
	gst_element_set_state(data->pipeline, GST_STATE_NULL);
}

```

析构的时候释放资源:
```
ShortAudio::~ShortAudio()
{
	gst_element_set_state(data->pipeline, GST_STATE_NULL);
	g_object_unref(data->pipeline);
	g_free(data);
}
```

# 预览并录制保存mp4 视频

头文件如下:
```
#include <QWidget>
#include "gst/gst.h"
#include <gst/video/videooverlay.h>

typedef struct
{
	GstElement *pipeline_view, *view_src, *view_sink;
	GstElement *pipeline;
	GstElement *videosrc, *videorate, *videoconvert, *encqueue, *videoenc, *videodec, *videoparse;
	GstElement *videotee, *sinkqueue, *videosinkconv, *videosink;
	GstElement *audiosrc, *audiorate, *audioconvert, *audioenc;
	GstElement *muxer, *filesink, *fakesink;
}ShortVideoData;

class ShortVideo
{
public:
	explicit ShortVideo(QString path, QWidget *window);
	~ShortVideo();

	void start();
	void stop();

private:
	ShortVideoData *data;
};

```

首先建立一个用于预览的通道，直接连接视频源和播放器，因为需要指定播放器播放的界面，所以这里用d3dvideosink作为播放元件，后面会指定界面:
```
data->pipeline_view = gst_pipeline_new("video_view");

data->view_src = gst_element_factory_make("autovideosrc", "view_src");
data->view_sink = gst_element_factory_make("d3dvideosink", "view_sink");

gst_bin_add_many(GST_BIN(data->pipeline_view), data->view_src, data->view_sink, NULL);
if (!gst_element_link_many(data->view_src, data->view_sink, NULL))
{
	qDebug() << "fail to link view";
}
```

录制通道视频源需要分成两路，一路用于预览，一路用于生成视频文件，需要注意的是如果直接把数据源分两路，会使通道卡死，原因未知，所以先编码成h264然后用tee元件分成两路:
```
data->videosrc = gst_element_factory_make("autovideosrc", "shortvideo_src");
data->videoconvert = gst_element_factory_make("videoconvert", "shortvideo_conv2");
data->videoenc = gst_element_factory_make("openh264enc", "shortvideo_enc");
data->videoparse = gst_element_factory_make("h264parse", "shortvideo_parse");
data->videotee = gst_element_factory_make("tee", "shortvideo_tee");
```
建立预览通道:
```
data->sinkqueue = gst_element_factory_make("queue", "shortvideo_sinkqueue");
data->videodec = gst_element_factory_make("avdec_h264", "shortvideo_decodebin");
data->videosink = gst_element_factory_make("d3dvideosink", "shortvideo_sink");
```
编码通道需要使用mp4mux与音频混合成mp4视频文件，音频使用的是voaacenc元件:
```
data->audiosrc = gst_element_factory_make("autoaudiosrc", "shortaudio_src");
data->audiorate = gst_element_factory_make("audiorate", "shortaudio_rate");
data->audioconvert = gst_element_factory_make("audioconvert", "shortaudio_conv");
data->audioenc = gst_element_factory_make("voaacenc", "shortaudio_enc");

data->muxer = gst_element_factory_make("mp4mux", "short_mm");
data->filesink = gst_element_factory_make("filesink", "short_fs");

data->fakesink = gst_element_factory_make("fakesink", "short_video_fakesink");
```

设置文件路径
```
g_object_set(data->filesink, "location", path.toStdString().c_str(), NULL);
```
将录制通道所有元件添加到pipeline:
```
gst_bin_add_many(GST_BIN(data->pipeline), data->videosrc, data->videoconvert, data->videoenc, data->videoparse,
	data->videotee, data->sinkqueue, data->videodec, data->videosink, data->encqueue,
	data->audiosrc, data->audiorate, data->audioconvert, data->audioenc, data->muxer, data->filesink, NULL);
```
连接视频源通道:
```
if (!gst_element_link_many(data->videosrc, data->videoconvert, data->videoenc, data->videoparse, data->videotee, NULL))
{
	qDebug() << "fail to link video";
}
```
连接预览通道:
```
GstPad *srcpad, *sinkpad;
srcpad = gst_element_get_request_pad(data->videotee, "src_0");
sinkpad = gst_element_get_static_pad(data->sinkqueue, "sink");
if (gst_pad_link(srcpad, sinkpad) != GST_PAD_LINK_OK)
	qDebug() << "fail to link tee and sendconv";
gst_object_unref(srcpad);
gst_object_unref(sinkpad);

if (!gst_element_link_many(data->sinkqueue, data->videodec, data->videosink, NULL))
{
	qDebug() << "fail to link video";
}
```
连接录制通道
```
srcpad = gst_element_get_request_pad(data->videotee, "src_1");
sinkpad = gst_element_get_static_pad(data->encqueue, "sink");
if (gst_pad_link(srcpad, sinkpad) != GST_PAD_LINK_OK)
	qDebug() << "fail to link tee and playsink";
gst_object_unref(srcpad);
gst_object_unref(sinkpad);

if (!gst_element_link_many(data->audiosrc, data->audiorate, data->audioconvert, data->audioenc, NULL))
{
	qDebug() << "fail to link audio";
}
```
将视频和音频混合:
```
srcpad = gst_element_get_static_pad(data->encqueue, "src");
sinkpad = gst_element_get_request_pad(data->muxer, "video_0");
if (gst_pad_link(srcpad, sinkpad) != GST_PAD_LINK_OK)
	qDebug() << "fail to link video and muxer";
gst_object_unref(srcpad);
gst_object_unref(sinkpad);

srcpad = gst_element_get_static_pad(data->audioenc, "src");
sinkpad = gst_element_get_request_pad(data->muxer, "audio_0");
if (gst_pad_link(srcpad, sinkpad) != GST_PAD_LINK_OK)
	qDebug() << "fail to link audio and muxer";
gst_object_unref(srcpad);
gst_object_unref(sinkpad);

if (!gst_element_link_many(data->muxer, data->filesink, NULL))
{
	qDebug() << "fail to link file";
}
```
设置界面:
```
WId d3d = window->winId();
gst_video_overlay_set_window_handle(GST_VIDEO_OVERLAY(data->videosink), d3d);
gst_video_overlay_set_window_handle(GST_VIDEO_OVERLAY(data->view_sink), d3d);
```
启动预览:
```
gst_element_set_state(data->pipeline_view, GST_STATE_PLAYING);
```

播放时关闭预览，开启录制，注意需要在结束后给出一个结束信号，并且稍微延时，不然文件不能播放，停止时全部关闭:
```
void ShortVideo::start()
{
	gst_element_set_state(data->pipeline_view, GST_STATE_NULL);
	gst_element_set_state(data->pipeline, GST_STATE_PLAYING);
}

void ShortVideo::stop()
{
	gst_element_set_state(data->pipeline_view, GST_STATE_NULL);
	gst_element_send_event(data->pipeline, gst_event_new_eos());
	_sleep(1000);
	gst_element_set_state(data->pipeline, GST_STATE_NULL);
}

```
释放资源:
```
ShortVideo::~ShortVideo()
{
	gst_element_set_state(data->pipeline_view, GST_STATE_NULL);
	gst_element_set_state(data->pipeline, GST_STATE_NULL);
	g_object_unref(data->pipeline);
	g_object_unref(data->pipeline_view);
	g_free(data);
}
```

# 取MP4视频的其中一帧图像并保存

头文件如下:
```
#include <QImage>
#include <QString>
#include "gst/gst.h"
#include "string.h"
#include <gst/app/gstappsink.h>

typedef struct
{
	std::string video_view_path;
	GstElement *pipeline;
	GstElement *filesrc, *qtdemux, *queue, *videodec, *videobox, *videoconv, *appsink;
}VideoImageViewData;

class VideoImageView
{
public:
	explicit VideoImageView(QString path);
	~VideoImageView();

private:
	static void new_video_sample_pad(GstElement *element, GstPad *pad, gpointer data);
	static GstFlowReturn new_image_from_appsink(GstElement * elt, VideoImageViewData * data);

private:
	VideoImageViewData *data;
};

```

用到如下元件:

+ filesrc
+ qtdemux
+ queue
+ avdec_h264
+ videobox
+ videoconvert
+ appesink

初始化元件，设置图片保存路径以及视频源路径:
```
data->video_view_path = (path + ".jpg").toStdString();
data->pipeline = gst_pipeline_new("video_sample");
data->filesrc = gst_element_factory_make("filesrc", "video_sample_file_src");
data->qtdemux = gst_element_factory_make("qtdemux", "video_sample_demux");
data->queue = gst_element_factory_make("queue", "video_sample_queue");
data->videodec = gst_element_factory_make("avdec_h264", "video_sample_videodec");
data->videobox = gst_element_factory_make("videobox", "video_sample_videobox");
data->videoconv = gst_element_factory_make("videoconvert", "video_sample_conv");
data->appsink = gst_element_factory_make("appsink", "video_sample_appsink");
g_object_set(data->filesrc,
	"location", path.toStdString().c_str(),
	NULL);
```

对视频进行裁剪:
```
g_object_set(G_OBJECT(data->videobox), "autocrop", TRUE, NULL);
```

定义appsink的caps:
```
#define VIDEO_SAMPLE_CONV_CAPS "video/x-raw, format=(string)RGB16, width=(int)120, height=(int)160"
```

设置appsink的caps以及数据获取方式:
```
GstCaps *convert_caps;
convert_caps = gst_caps_from_string(VIDEO_SAMPLE_CONV_CAPS);
g_object_set(G_OBJECT(data->appsink), "emit-signals", TRUE, "sync", FALSE, NULL);
g_object_set(G_OBJECT(data->videoconv), "caps", convert_caps, NULL);
g_object_set(G_OBJECT(data->appsink), "caps", convert_caps, NULL);
gst_caps_unref(convert_caps);
```

分别连接源和视频处理通道，qtdemux需要以回调的方式连接:
```
if (!gst_element_link_many(data->filesrc, data->qtdemux, NULL))
	qDebug() << "fail to link filesrc qtdemux";

if (!gst_element_link_many(data->queue, data->videodec, data->videobox, NULL))
	qDebug() << "fail to link queue videodec videobox";
```

定义videobox和videoconvert之间的caps，连接后面的元件:
```
GstCaps *caps_box;
caps_box = gst_caps_from_string(VIDEO_SAMPLE_BOX_CAPS);
if (!gst_element_link_filtered(data->videobox, data->videoconv, caps_box))
	qDebug() << "Failed to link caps";
gst_caps_unref(caps_box);
if (!gst_element_link_many(data->videoconv, data->appsink, NULL))
	qDebug() << "fail to link videoconv appsink";
```

定义qtdemux连接回调，因为只需要视频，当出现新连接，如果是视频sink则进行连接:
```
void VideoImageView::new_video_sample_pad(GstElement *, GstPad *pad, gpointer data)
{
	GstPad *sinkpad;
	sinkpad = gst_element_get_static_pad(static_cast<VideoImageViewData*>(data)->queue, "sink");
	if (gst_pad_link(pad, sinkpad) != GST_PAD_LINK_OK)
		qDebug() << "fail to link qtdemux and videodec";
	gst_object_unref(sinkpad);
}
```

绑定pad-added信号和回调:
```
g_signal_connect(data->qtdemux, "pad-added", G_CALLBACK(new_video_sample_pad), data);
```

定义appsink数据回调，当有新数据时，保存为图片:
```
GstFlowReturn
VideoImageView::new_image_from_appsink(GstElement * elt, VideoImageViewData * data)
{
	GstSample *sample;
	GstBuffer *buffer;
	GstMapInfo map;
	sample = gst_app_sink_pull_sample(GST_APP_SINK(elt));
	if (!sample)
		return GstFlowReturn(0);
	buffer = gst_sample_get_buffer(sample);
	gst_buffer_map(buffer, &map, GST_MAP_READ);
	gst_sample_unref(sample);

	QImage image = QImage(map.data, VIDEO_SAMPLE_WIDTH, VIDEO_SAMPLE_HEIGHT, QImage::Format_RGB16);
	image.save(QString((data->video_view_path).c_str()), "JPG");

	gst_buffer_unmap(buffer, &map);

	return GstFlowReturn(0);
}
```

绑定appsin的信号和回调函数:
```
g_signal_connect(data->qtdemux, "pad-added", G_CALLBACK(new_video_sample_pad), data);
g_signal_connect(data->appsink, "new-sample", G_CALLBACK(new_image_from_appsink), data);
```

直接启动:
```
gst_element_set_state(data->pipeline, GST_STATE_PLAYING);
```

析构时停止:
```
VideoImageView::~VideoImageView()
{
	gst_element_set_state(data->pipeline, GST_STATE_NULL);
	g_object_unref(data->pipeline);
	g_free(data);
}
```

# 播放一个mp4文件

头文件如下:
```
#include <QWidget>
#include <QDir>
#include <thread>
#include <QDebug>
#include "gst/gst.h"
#include <gst/video/videooverlay.h>
#include <gst/app/gstappsrc.h>
#include <gst/app/gstappsink.h>


typedef struct
{
	GMainLoop *loop;
	GstElement *playbin, *videosink;
	GstElement *pipeline;
	GstElement *filesrc, *qtdemux;
	GstElement *video_queue, *video_dec, *video_flip, *video_conv, *video_sink;
	GstElement *audio_queue, *audio_dec, *audio_conv, *audio_sink;
}VideoPlayerData;

class VideoPlayer
{
public:
	explicit VideoPlayer(QString path, QWidget* widget);
	~VideoPlayer();

	void start();
	void stop();
	bool is_running();
	gint64 get_pos();
	gint64 get_dur();

private:
	void run();
	static gboolean call_back(GstBus *, GstMessage *msg, gpointer data);
	static void new_player_pad(GstElement *, GstPad *pad, gpointer data);

private:
	VideoPlayerData *data;
	std::thread t;
};
```

需要用到的元件:

+ filesrc
+ qtdemux
+ queue
+ avdec_h264
+ videoflip
+ videoconvert
+ d3dvideosink
+ queue
+ avdec_aac
+ audioconvert
+ autoaudiosink

初始化元件:
```
data->loop = g_main_loop_new(NULL, FALSE);

data->pipeline = gst_pipeline_new("player_pipeline");
data->filesrc = gst_element_factory_make("filesrc", "player_filesrc");
data->qtdemux = gst_element_factory_make("qtdemux", "player_qtdemux");

data->video_queue = gst_element_factory_make("queue", "player_video_queue");
data->video_dec = gst_element_factory_make("avdec_h264", "player_video_dec");
data->video_flip = gst_element_factory_make("videoflip", "player_video_flip");
data->video_conv = gst_element_factory_make("videoconvert", "player_video_convert");
data->video_sink = gst_element_factory_make("d3dvideosink", "player_video_sink");

data->audio_queue = gst_element_factory_make("queue", "player_audio_queue");
data->audio_dec = gst_element_factory_make("avdec_aac", "player_audio_dec");
data->audio_conv = gst_element_factory_make("audioconvert", "player_audio_conv");
data->audio_sink = gst_element_factory_make("autoaudiosink", "player_audio_sink");
```
这里使用了mainloop，但应该是不需要的，因为并没有需要检测停止信号。

设置文件路径，设置视频自动旋转，这样videoflip会自动检测视频是否需要旋转，并进行旋转操作。
```
g_object_set(data->filesrc, "location", path.toStdString().c_str(), NULL);
g_object_set(data->video_flip, "method", 8, NULL);
```

qtdemux需要用信号的方式连接，因此先连接几个通道:
```
gst_bin_add_many(GST_BIN(data->pipeline), data->filesrc, data->qtdemux, 
	data->video_queue, data->video_dec, data->video_flip, data->video_conv, data->video_sink,
	data->audio_queue, data->audio_dec, data->audio_conv, data->audio_sink, NULL);

if (!gst_element_link_many(data->filesrc, data->qtdemux, NULL))
	qDebug() << "fail to link filesrc qtdemux";

if (!gst_element_link_many(data->video_queue, data->video_dec, data->video_flip, 
	data->video_conv, data->video_sink, NULL))
	qDebug() << "fail to link player video queue";

if (!gst_element_link_many(data->audio_queue, data->audio_dec, data->audio_conv,
	data->audio_sink, NULL))
	qDebug() << "fail to link player audio queue";
```

定义qtdemux的连接回调函数:
```
void VideoPlayer::new_player_pad(GstElement *, GstPad *pad, gpointer data)
{
	auto name = gst_object_get_name(GST_OBJECT_CAST(pad));

	if (QString(static_cast<char*>(name)) == "video_0")
	{
		GstPad *sinkpad;
		sinkpad = gst_element_get_static_pad(static_cast<VideoPlayerData*>(data)->video_queue, "sink");
		if (gst_pad_link(pad, sinkpad) != GST_PAD_LINK_OK)
			qDebug() << "fail to link qtdemux and videodec";
		gst_object_unref(sinkpad);
	}
	else
	{
		GstPad *sinkpad;
		sinkpad = gst_element_get_static_pad(static_cast<VideoPlayerData*>(data)->audio_queue, "sink");
		if (gst_pad_link(pad, sinkpad) != GST_PAD_LINK_OK)
			qDebug() << "fail to link qtdemux and audiodec";
		gst_object_unref(sinkpad);
	}
}
```

绑定信号:
```
g_signal_connect(data->qtdemux, "pad-added", G_CALLBACK(new_player_pad), data);
```

设置界面:
```
WId d3d = widget->winId();
gst_video_overlay_set_window_handle(GST_VIDEO_OVERLAY(data->video_sink), d3d);
```

提供控制接口:
```
void VideoPlayer::start()
{
	t = std::thread(&VideoPlayer::run, this);
}

void VideoPlayer::stop()
{
	gst_element_set_state(data->pipeline, GST_STATE_NULL);
	g_main_loop_quit(data->loop);
	if (t.joinable())
		t.join();
}

bool VideoPlayer::is_running()
{
	GstState state;
	gst_element_get_state(data->pipeline, &state, NULL, GstClockTime(1000000));
	return state == GST_STATE_PLAYING;
}

void VideoPlayer::run()
{
	gst_element_set_state(data->pipeline, GST_STATE_PLAYING);
	g_main_loop_run(data->loop);
}
```

提供获取播放信息的接口，分别是播放的位置，以及总长度:
```
gint64 VideoPlayer::get_pos()
{
	GstFormat fmt = GST_FORMAT_TIME;
	gint64 res;
	gst_element_query_position(data->pipeline, fmt, &res);
	return res;
}

gint64 VideoPlayer::get_dur()
{
	GstFormat fmt = GST_FORMAT_TIME;
	gint64 res;
	gst_element_query_duration(data->pipeline, fmt, &res);
	return res;
}
```

# TCP发送音频流

这里继承了QThread，头文件如下:
```
#include <QThread>
#include <QDebug>
#include <stdio.h>
#include <stdlib.h>
#include <winsock2.h>
#include "gst/gst.h"
#include <gst/app/gstappsrc.h>
#include <gst/app/gstappsink.h>

typedef struct
{
	GMainLoop *loop;
	GstElement *src, *convert, *encoder, *sink;
	GstElement *pipeline;
	SOCKET sock;
	SOCKADDR_IN address;
} AudioSenderData;


class AudioSender : public QThread
{
    Q_OBJECT

public:
    explicit AudioSender(SOCKET &sk, QString host);
    ~AudioSender();

	void run();
	static GstFlowReturn
		on_new_sample_from_sink(GstElement * elt, AudioSenderData * data);
	static int32_t	bytes_count;

private:
	AudioSenderData *data;
};
```

用到的元件如下:

+ autoaudiosrc
+ audioconvert
+ voaacenc
+ appsink

初始化元件:
```
data->sock = sk;
data->pipeline = gst_pipeline_new("audio_pipeline_sender");
data->src = gst_element_factory_make("autoaudiosrc", "audio_src_sender");
data->convert = gst_element_factory_make("audioconvert", "audio_conv_sender");
data->encoder = gst_element_factory_make("voaacenc", "audio_enc_sender");
data->sink = gst_element_factory_make("appsink", "audio_sink_sender");
```

设置一下码率:
```
g_object_set(G_OBJECT(data->encoder), "bitrate", bitrate, NULL);
```

设置appsink以信号方式获取数据:
```
g_object_set(G_OBJECT(data->sink), "emit-signals", TRUE, "sync", FALSE, NULL);
```

定义一个处理数据的回调，取出数据之后通过TCP发送:
```
GstFlowReturn AudioSender::on_new_sample_from_sink(GstElement * elt, AudioSenderData * data)
{
	GstSample *sample;
	GstBuffer *buffer;
	GstMapInfo map;
	sample = gst_app_sink_pull_sample(GST_APP_SINK(elt));
	if (!sample)
		return GstFlowReturn(0);
	buffer = gst_sample_get_buffer(sample);
	gst_buffer_map(buffer, &map, GST_MAP_READ);

	gst_sample_unref(sample);

	gsize map_size;
	gsize total_size;
	map_size = map.size;
	total_size = map_size + 2;

	guint8 size_buffer[6];
	size_buffer[0] = total_size;
	size_buffer[1] = total_size >> 8;
	size_buffer[2] = total_size >> 16;
	size_buffer[3] = total_size >> 24;

	size_buffer[4] = 0;
	size_buffer[5] = 0;

	gsize send_size = 0;
	gsize server_send;
	while (send_size < 6)
	{
		server_send = send(data->sock, (char *)size_buffer + send_size, 
			6 - send_size, 0);
		send_size += server_send;
	}

	send_size = 0;
	while (send_size < map_size)
	{
		server_send = send(data->sock, (char *)(map.data) + send_size,
			map_size - send_size, 0);
		send_size += server_send;
	}

	gst_buffer_unmap(buffer, &map);

	return GstFlowReturn(0);
}
```

设置appsink回调:
```
g_signal_connect(data->sink, "new-sample",
	G_CALLBACK(on_new_sample_from_sink), data);
```

设置appsink caps:
```
#define AUDIO_AAC_SENDER "audio/mpeg, mpegversion=(int)4, channels=(int)1, rate=(int)8000, base-profile=(string)lc, stream-format=(string)adts"


GstCaps *caps;
caps = gst_caps_from_string(AUDIO_AAC_SENDER);
g_object_set(G_OBJECT(data->sink), "caps", caps, NULL);
gst_caps_unref(caps);
```

连接通道
```
gst_bin_add_many(GST_BIN(data->pipeline), data->src, data->convert, data->encoder,
	data->sink, NULL);

if (!gst_element_link_many(data->src, data->convert, data->encoder, NULL))
	qDebug() << "fail to link audio sender pipeline";
```

连接encoder和sink时插入caps:
```
GstCaps *caps_enc;
caps_enc = gst_caps_from_string(AUDIO_AAC_SENDER);
if (!gst_element_link_filtered(data->encoder, data->sink, caps_enc))
	qDebug() << "Failed to link caps";
gst_caps_unref(caps_enc);
```

启动接口以及释放资源:
```
AudioSender::~AudioSender()
{
	gst_element_set_state(data->pipeline, GST_STATE_NULL);
	g_object_unref(data->pipeline);
	g_free(data);
}

void AudioSender::run()
{
	gst_element_set_state(data->pipeline, GST_STATE_PLAYING);
}
```

# TCP 接收播放音频流

继承QThread，头文件:
```
#include <QThread>
#include <QDebug>
#include <stdio.h>
#include <stdlib.h>
#include <winsock2.h>
#include "gst/gst.h"
#include <gst/app/gstappsrc.h>
#include <gst/app/gstappsink.h>

typedef struct
{
	GstElement *pipeline, *appsrc, *parse, *decoder, *conv, *sink;
	SOCKET sock;
} AudioRecData;

namespace Ui {
class AudioRec;
}

class AudioRec : public QThread
{
    Q_OBJECT

public:
    explicit AudioRec(SOCKET &sk);
    ~AudioRec();

	void run();
	void prepare_data();

	static long cout_miss;
	static long cout_rec;

private:
	AudioRecData *data;
	int flag;
	int last_number = -1;
};
```

使用元件:

+ appsrc
+ aacparse
+ avdec_aac
+ audioconvert
+ autoaudiosink

初始化元件:
```
data->sock = sk;
data->pipeline = gst_pipeline_new("audio_piprec");
data->appsrc = gst_element_factory_make("appsrc", "audio_srcrec");
data->parse = gst_element_factory_make("aacparse", "audio_parserec");
data->decoder = gst_element_factory_make("avdec_aac", "audio_decrec");
data->conv = gst_element_factory_make("audioconvert", "audio_convrec");
data->sink = gst_element_factory_make("autoaudiosink", "audio_sinkrec");
```

设置appsrc的caps:
```
#define AUDIO_AAC_REC "audio/mpeg, mpegversion=(int)4, channels=(int)1, rate=(int)8000, base-profile=(string)lc, stream-format=(string)adts"

GstCaps *caps;
caps = gst_caps_from_string(AUDIO_AAC_REC);
g_object_set(G_OBJECT(data->appsrc), "caps", caps, NULL);
gst_caps_unref(caps);
```

设置block，以循环push的方式往通道传入参数:
```
g_object_set(data->appsrc,
	"format", GST_FORMAT_TIME,
	"block", TRUE,
	NULL);
```

设置sync参数，在丢包时通道不会卡住:
```
g_object_set(G_OBJECT(data->sink), "sync", FALSE, NULL);
```

连接:
```
gst_bin_add_many(GST_BIN(data->pipeline), data->appsrc, data->parse, data->decoder, data->conv, data->sink, NULL);

if (!gst_element_link_many(data->appsrc, data->parse, NULL))
	qDebug() << "fail to link audio receive pipeline";
```

连接parse和decoder时指定caps:
```
GstCaps *caps_dec;
caps_dec = gst_caps_from_string(AUDIO_DEC_REC);
if (!gst_element_link_filtered(data->parse, data->decoder, caps_dec))
	qDebug() << "Failed to link caps";
gst_caps_unref(caps_dec);

if (!gst_element_link_many(data->decoder, data->conv, data->sink, NULL))
	qDebug() << "fail to link audio receive pipeline";
```

循环从tcp读取数据并传入通道:
```
void AudioRec::prepare_data()
{
	guint8 size_buffer[6];
	gsize rec_size = 0;
	gsize client_rec_size;

	while (rec_size < 6)
	{
		client_rec_size = recv(data->sock, (char *)size_buffer + rec_size,
			6 - rec_size, 0);
		if (client_rec_size > 6 || client_rec_size == SOCKET_ERROR)
		{
			flag = 0;
			return;
		}
		rec_size += client_rec_size;
	}

	gsize total_size;
	total_size = size_buffer[0]
		+ (size_buffer[1] << 8)
		+ (size_buffer[2] << 16)
		+ (size_buffer[3] << 24);

	gsize map_size;
	map_size = total_size - 2;

	//qDebug() << "audio recive size: " << total_size;

	if (map_size > USHRT_MAX || total_size < 2)
	{
		qDebug() << "audio size is not valid";
		flag = 0;
		return;
	}

	char *frame_buffer = (char *)g_malloc(map_size * sizeof(char));

	rec_size = 0;
	while (rec_size < map_size)
	{
		client_rec_size = recv(data->sock, &frame_buffer[rec_size],
			map_size - rec_size, 0);
		if (client_rec_size > map_size || client_rec_size == SOCKET_ERROR)
		{
			flag = 0;
			return;
		}
		rec_size += client_rec_size;
	}

	GstBuffer *buffer;
	buffer = gst_buffer_new_wrapped((gpointer)(frame_buffer), map_size * sizeof(char));

	gst_app_src_push_buffer(GST_APP_SRC(data->appsrc), buffer);
}

void AudioRec::run()
{
	gst_element_set_state(data->pipeline, GST_STATE_PLAYING);
	flag = 1;
	while (flag)
	{
		prepare_data();
	}
}
```

# TCP 发送视频流

头文件如下:
```
#include <QThread>
#include <QDebug>
#include <stdio.h>
#include <stdlib.h>
#include <winsock2.h>
#include "gst/gst.h"
#include <gst/app/gstappsrc.h>
#include <gst/app/gstappsink.h>
#include <gst/video/videooverlay.h>

typedef struct
{
	GMainLoop *loop;
	GstElement *videosrc, *videorate, *videobox, *videotee, *sendqueue, *playqueue, 
		*sendconv, *encoder, *sendsink, *playsrc, *playconv, *playsink;
	GstElement *pipeline;
	SOCKET sock;
	SOCKADDR_IN address;
} VideoSenderData;


class VideoSender : public QThread
{
    Q_OBJECT

public:
    explicit VideoSender(SOCKET& sk, QString host, QWidget* w);
    ~VideoSender();

    void run();
	static GstFlowReturn
		on_new_sample_from_sink(GstElement * elt, VideoSenderData * data);
	static int32_t	bytes_count;

private:
	VideoSenderData *data;

	QString phone_ip;
};
```

用到的元件:

+ autovideosrc
+ videorate
+ videobox
+ tee
+ queue
+ videoconvert
+ openh264enc
+ appsink
+ d3dvideosink

初始化元件:
```
data->sock = sk;
data->pipeline = gst_pipeline_new("pipeline_src_sender");

data->videosrc = gst_element_factory_make("autovideosrc", "source_sender");
data->videorate = gst_element_factory_make("videorate", "rate_sender");
data->videobox = gst_element_factory_make("videobox", "box_sender");
data->videotee = gst_element_factory_make("tee", "selfsink_sender");

data->sendqueue = gst_element_factory_make("queue", "sendqueue_sender");
data->sendconv = gst_element_factory_make("videoconvert", "convert2_sender");
data->encoder = gst_element_factory_make("openh264enc", "encoder_sender");
data->sendsink = gst_element_factory_make("appsink", "appsink_sender");

data->playqueue = gst_element_factory_make("queue", "playqueue_sender");
data->playconv = gst_element_factory_make("videoconvert", "convert3_sender");
data->playsink = gst_element_factory_make("d3dvideosink", "d3d_sender");
```

设置码率，根据码率选择不同的分辨率的caps，设置appsink的caps:
```
#define VIDEO_CAPS_SENDER_480 "video/x-h264, stream-format=(string)byte-stream, alignment=(string)au, profile=(string)baseline, width=(int)360, height=(int)480, framerate=(fraction)30/1"

#define VIDEO_CAPS_SENDER_320 "video/x-h264, stream-format=(string)byte-stream, alignment=(string)au, profile=(string)baseline, width=(int)240, height=(int)320, framerate=(fraction)30/1"



if (bitrate <= 128000)
{
	caps = gst_caps_from_string(VIDEO_CAPS_SENDER_320);
	bitrate *= 0.8;
}
else
{
	caps = gst_caps_from_string(VIDEO_CAPS_SENDER_480);
}

g_object_set(G_OBJECT(data->encoder),
	"bitrate", bitrate,
	"rate-control", 1,
	"max-bitrate", bitrate,
	"enable-frame-skip", true,
	NULL);

g_object_set(G_OBJECT(data->sendsink), "emit-signals", TRUE, "sync", FALSE, NULL);
g_object_set(G_OBJECT(data->sendsink), "caps", caps, NULL);
gst_caps_unref(caps);
```

设置裁剪方式:
```
g_object_set(G_OBJECT(data->videobox), "autocrop", TRUE, NULL);
```

定义回调函数:
```
GstFlowReturn 
VideoSender::on_new_sample_from_sink(GstElement * elt, VideoSenderData * data)
{
	GstSample *sample;
	GstBuffer *buffer;
	GstMapInfo map;
	sample = gst_app_sink_pull_sample(GST_APP_SINK(elt));
	if (!sample)
		return GstFlowReturn(0);
	buffer = gst_sample_get_buffer(sample);
	gst_buffer_map(buffer, &map, GST_MAP_READ);
	gst_sample_unref(sample);

	gsize map_size;
	gsize total_size;
	map_size = map.size;
	total_size = map_size + 2;

	guint8 size_buffer[6];
	size_buffer[0] = total_size;
	size_buffer[1] = total_size >> 8;
	size_buffer[2] = total_size >> 16;
	size_buffer[3] = total_size >> 24;

	size_buffer[4] = 0;
	size_buffer[5] = 0;

	gsize send_size = 0;
	gsize server_send;
	while (send_size < 6)
	{
		server_send = send(data->sock, (char *)size_buffer + send_size,
			6 - send_size, 0);
		send_size += server_send;
	}
	send_size = 0;
	while (send_size < map_size)
	{
		server_send = send(data->sock, (char *)(map.data) + send_size,
			map_size - send_size, 0);
		send_size += server_send;
	}

	gst_buffer_unmap(buffer, &map);

	return GstFlowReturn(0);
}
```

appsink信号绑定回调:
```
g_signal_connect(data->sendsink, "new-sample",
	G_CALLBACK(on_new_sample_from_sink), data);
```

视频源分成两路，一路通过appsink由TCP发送，一路通过播放元件预览:
```
GstPad *srcpad, *sinkpad;
srcpad = gst_element_get_request_pad(data->videotee, "src_0");
sinkpad = gst_element_get_static_pad(data->sendqueue, "sink");
if (gst_pad_link(srcpad, sinkpad) != GST_PAD_LINK_OK)
	qDebug() << "fail to link tee and sendconv";
gst_object_unref(srcpad);
gst_object_unref(sinkpad);

srcpad = gst_element_get_request_pad(data->videotee, "src_1");
sinkpad = gst_element_get_static_pad(data->playqueue, "sink");
if (gst_pad_link(srcpad, sinkpad) != GST_PAD_LINK_OK)
	qDebug() << "fail to link tee and playsink";
gst_object_unref(srcpad);
gst_object_unref(sinkpad);

if (!gst_element_link_many(data->sendqueue, data->sendconv, data->encoder, data->sendsink, NULL))
	qDebug() << "fail to link sender pipeline";

if (!gst_element_link_many(data->playqueue, data->playconv, data->playsink, NULL))
	qDebug() << "fail to link sender pipeline";
```

设置预览界面:
```
WId d3d = w->winId();
gst_video_overlay_set_window_handle(GST_VIDEO_OVERLAY(data->playsink), d3d);
```

析构函数释放资源，提供启动接口:
```
VideoSender::~VideoSender()
{
	gst_element_set_state(data->pipeline, GST_STATE_NULL);
	g_object_unref(data->pipeline);
	g_free(data);
}

void VideoSender::run()
{
	//gst_element_set_state(data->sendsink, GST_STATE_PLAYING);
	gst_element_set_state(data->pipeline, GST_STATE_PLAYING);
}
```

# TCP 接收视频流并显示

头文件如下:
```
#include <QThread>
#include <QDebug>
#include <stdio.h>
#include <stdlib.h>
#include <winsock2.h>
#include <QWidget>
#include "gst/gst.h"
#include <gst/app/gstappsrc.h>
#include <gst/app/gstappsink.h>
#include <gst/video/videooverlay.h>
#include <gst/video/gstvideofilter.h>
#include <gst/video/video.h>

typedef struct
{
	GstElement *pipeline, *appsrc, *decoder, *videoflip, *conv, *videosink;
	SOCKET sock;
} VideoRecData;


class VideoRec : public QThread
{
    Q_OBJECT

public:
    explicit VideoRec(SOCKET &sk, QWidget* w);
    ~VideoRec();

    void run();
	void prepare_data();
	void flip();
	GstState get_state();

	static long cout_miss;
	static long cout_rec;

private:
	VideoRecData *data;
	int flag;
	int last_number = -1;
	int rotate_index = 0;
	std::vector<const char*> rotate{ "none", "clockwise", "rotate-180", "counterclockwise" };
};
```

用到的元件:

+ appsrc
+ openh264dec
+ videoflip
+ videoconvert
+ d3dvideosink

初始化元件:
```
data->sock = sk;
data->pipeline = gst_pipeline_new("video_pipeline_rec");
data->appsrc = gst_element_factory_make("appsrc", "video_srcrec");
data->decoder = gst_element_factory_make("openh264dec", "video_decrec");
data->videoflip = gst_element_factory_make("videoflip", "video_flip");
data->conv = gst_element_factory_make("videoconvert", "video_convrec");
data->videosink = gst_element_factory_make("d3dvideosink", "video_sinkrec");
```

设置sync参数，否则丢包会导致播放失败:
```
g_object_set(G_OBJECT(data->videosink), "sync", FALSE, NULL);
```

设置appsrc的caps:
```
#define VIDEO_CAPS_REC "video/x-h264, stream-format=(string)byte-stream, alignment=(string)au, profile=(string)baseline"


GstCaps *caps;
caps = gst_caps_from_string(VIDEO_CAPS_REC);
g_object_set(G_OBJECT(data->appsrc), "caps", caps, NULL);
gst_caps_unref(caps);
```

appsink的blok属性设为true，以循环push的方式读取数据并传入通道:
```
g_object_set(data->appsrc,
	"format", GST_FORMAT_TIME,
	"blok", TRUE,
	NULL);
```

连接:
```
gst_bin_add_many(GST_BIN(data->pipeline), data->appsrc,
	data->decoder, data->videoflip, data->conv, data->videosink, NULL);

if (!gst_element_link_many(data->appsrc, data->decoder, data->videoflip, data->conv, data->videosink, NULL))
	qDebug() << "fail to link video receiver";
```

设置界面:
```
WId d3d = w->winId();
gst_video_overlay_set_window_handle(GST_VIDEO_OVERLAY(data->videosink), d3d);
```

通过设置videoflip的属性实现旋转图像，各个角度的属性在头文件定义:
```
void VideoRec::flip()
{
	rotate_index += 1;
	rotate_index %= 4;
	gst_util_set_object_arg(G_OBJECT(data->videoflip), "method", rotate[rotate_index]);
}
```

循环读取数据:
```
void VideoRec::prepare_data()
{
	guint8 size_buffer[6];
	gsize rec_size = 0;
	gsize client_rec_size;

	while (rec_size < 6)
	{
		client_rec_size = recv(data->sock, (char *)size_buffer + rec_size,
			6 - rec_size, 0);
		if (client_rec_size > 6 || client_rec_size == SOCKET_ERROR)
		{
			flag = 0;
			return;
		}
		rec_size += client_rec_size;
	}

	gsize total_size;
	total_size = size_buffer[0]
		+ (size_buffer[1] << 8)
		+ (size_buffer[2] << 16)
		+ (size_buffer[3] << 24);

	gsize map_size;
	map_size = total_size - 2;

	//qDebug() << "video recive size: " << total_size;

	if (map_size > USHRT_MAX || total_size < 2)
	{
		qDebug() << "video size is not valid";
		flag = 0;
		return;
	}

	char *frame_buffer = (char *)g_malloc(map_size * sizeof(char));

	rec_size = 0;
	while (rec_size < map_size)
	{
		client_rec_size = recv(data->sock, &frame_buffer[rec_size],
			map_size - rec_size, 0);
		if (client_rec_size > map_size || client_rec_size == SOCKET_ERROR)
		{
			flag = 0;
			return;
		}
		rec_size += client_rec_size;
	}

	GstBuffer *buffer;
	buffer = gst_buffer_new_wrapped((gpointer)(frame_buffer), map_size * sizeof(char));

	gst_app_src_push_buffer(GST_APP_SRC(data->appsrc), buffer);
}

void VideoRec::run()
{
	//gst_element_set_state(data->appsrc, GST_STATE_PLAYING);
	gst_element_set_state(data->pipeline, GST_STATE_PLAYING);
	flag = 1;
	while (flag)
	{
		prepare_data();
	}
}
```

析构函数:
```
VideoRec::~VideoRec()
{
	flag = 0;
	gst_element_set_state(data->pipeline, GST_STATE_NULL);
	g_object_unref(data->pipeline);
	g_free(data);
}

```
