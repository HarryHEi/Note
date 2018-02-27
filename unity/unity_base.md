---
title: unity basic
date: 2018-1-14 14:22:37
tags: [unity]
---

# 内置系统

地形系统：GameObject->3D Object->Terrain

# profiler

内置分析器用于捕获场景实时数据并分析性能。window -> profiler或者ctrl + 7。

# 导入模型

直接将相应的模型文件放到assets目录下导入。

# 材质

模型的好换取决于形状、材质和贴图。
一个模型可能有多个材质，一个材质可能对应多张贴图。

创建一个材质文件（材质球）： Create->Meterial

# 脚本

只有继承了MonoBehaviour类的才能附加到游戏物体上成为组件。

`Start()`方法在物体被创建时调用。

`Update()`方法每帧调用一次。


# Transform组件

transform是变换组件，定义了游戏对象的位置、旋转和缩放。
脚本中可以获取gameObject的transform进行变换操作。
```
gameObject.transform.Translate(0.1f, 0, 0);  //让物体沿着X轴移动0.1个单位
```

# Mesh Renderer组件

三维模型一般带有Mesh Renderer组件（网格渲染器）
从Mesh Filter（网格过滤器）获取几何形状，根据Transform位置进行渲染。

# 着色器

一个材质必须对应着一个Shader（着色器）

Standard Shader属性

Albedo： 物体表面基本颜色。
Metallic： 物体表面反光度，通常金属物体超过50%，大部分在90%以下，非金属在20%以下。
smoonthness： 值越大，物体越光滑。
normal map： 法线贴图。
Emission：自然光。

# prefab

prefab（预设）可以用于实例化对象，如果修改预设，所有实例均会被修改。

在脚本中创建一个预设对象：

```
public class TestCreatePrefab : MonoBehaviour
{
    public GameObject zombie;  //需要将一个预设对象放到脚本组件的该参数上

	// Use this for initialization
	void Start ()
    {
		
	}
	
	// Update is called once per frame
	void Update ()
    {
        float x = Random.Range(-10, 10);
        float y = Random.Range(-10, 10);
        float z = Random.Range(-10, 10);

        Vector3 pos = new Vector3(x, y, z);
        Instantiate(zombie, pos, Quaternion.identity);  //实例化一个预设对象
	}
}
```

# layer

layer(图层)横跨多个不同对象。

可以用图层指定对象绘制顺序或者照相机是否可见等。

通过对象的inspector窗口选择layer，通过add layer或者edit->project setting->Tags and Layers自定义layer。

# 场景视图

当需要移动场景时，右键长摁，通过`WASD`按钮可以进行"飞行模式"移动，通过按下`shift`加快移动速度。

按下鼠标右键，移动鼠标可以调整视角，按下`ctrl + alt`和鼠标右键，哟东鼠标可以调整距离。

如果想要移动视角到某个游戏对象，在Hierarchy(层级试图)窗口选中该对象，然后鼠标移动到场景窗口，按下`F`。

选中某个对象，然后按下`shift + F`，场景视图画面画面会跟着物体移动。

当选中某个照相机时，场景视图会出现照相机的预览。

按下`shift + space`全屏。

# 坐标系

2d游戏坐标系为表示左右方向的x和表示上下方向的y，3d坐标系增加了一个前后方向的z。

旋转方向沿坐标根据左右定律。

如果A是B的子对象，那么A坐标显示的是相对B的坐标。

# 声音

unity3D支持mp3、wav、ogg等多种音频格式。

要想声音生效，必须要有声音源和接收器两个组件，一个负责播放声音，另一个负责接收。
对于3D声音来说，如果音源和接收器相对位置发生变化，听到的声音也会随之改变。

AudioSource组件是一个音源组件，常用属性：

AudioClip： 要播放的声音。
Mute： 是否静音。
Bypass Effects： 是否打开音频特效。
Play On Awake： 是否自动播放。
Loop： 是否循环播放。
Volume： 声音大小。
Pitch： 播放速度，-3 ~ 3，1正常播放，大于1加速，小于1减速。

Audio Listener组件是接受组件，相机默认带有该属性。

## 一个简易的播放器

在一个物体上添加AudioSource组件并指定播放的声音，然后添加脚本：

```
public class AudioPlayer : MonoBehaviour
{
    void OnGUI()
    {
        AudioSource audio = GetComponent<AudioSource>();  //获取音源组件
        if(GUI.Button(new Rect(0, 0, 100, 50), "开始"))
        {
            audio.Play();
        }
        if(GUI.Button(new Rect(100, 0, 100, 50), "结束"))
        {
            audio.Stop();
        }
    }

    // Use this for initialization
    void Start ()
    {
		
	}
	
	// Update is called once per frame
	void Update ()
    {
		
	}
}

```
脚本中通过GetComponent方法获取对象的音源组件。
脚本会绘制GUI按钮，点击开始按钮播放，点击结束按钮停止播放。

# GUI

GUI绘图是绘制UI基础方法，只要在组件的OnGUI方法中调用绘图方法，就可以在游戏界面上绘制贴图、文字、按钮、滚动条等。
常见的GUI方法如下：

`GUI.Button`
绘制按钮，通常与if语句配合使用，它有两个参数。
第一个参数是按钮位置和大小；
第二个参数是按钮的文本。
```
if(GUI.Button(new Rect(0, 0, 100, 100), "按钮1"))
{
	Debug.Log("按钮1被按下");
}
```

`Gui.Label`
绘制文本，如：
```
GUI.Label(new Rect(0, 50, 100, 50), "这个是个播放器");
```

`Gui.Draw Texture`
绘制贴图，如：
```
GUI.DrawTexture(new Rect(0, 100, 100, 50), tex);
```
这里的tex参数是Texture类型。

`GUI.Box`： 绘制图形框
`GUI.Window`： 绘制窗口
`GUI.TextField`： 绘制输入框
`GUI.PasswordField`： 绘制密码输入框
`GUI.HorizontalScrollbar`： 绘制水平滚动条
`GUI.VerticalScrollbar`： 获知垂直滚动条

绘制一个登录框：
```
GUI.Box(new Rect(0, 0, 200, 120), "登录框");
GUI.Label(new Rect(10, 30, 50, 30), "用户名");
userName = GUI.TextField(new Rect(60, 30, 120, 20), userName);
GUI.Label(new Rect(10, 60, 50, 30), "密码");
password = GUI.PasswordField(new Rect(60, 60, 120, 20), password, '*');
if(GUI.Button(new Rect(60, 90, 50, 25), "登录"))
{
    Debug.Log("登录成功");
}
```

# 场景

游戏由多个场景（Scene）构成

File->New Scene创建场景，绘制并保存场景。

生成游戏时，只有添加的场景才会编入游戏。
File->Build Setting打开面板并添加场景。

旧版本使用`Application.LoadLevel("scene_2")`切换场景，
新版本使用`SceneManager.LoadScene("scene_2")`代替，
需要添加`using UnityEngine.SceneManagement`。

# 导出游戏

file->Build Setting选择平台

新版本中编辑器不会再附带平台模块。

点击Build可创建app文件。
