+++
date = "2015-05-20T18:49:15+08:00"
menu = "main"
tags = ["android"]
title = "Android Fragment 浅析"

+++

博客改自[http://blog.csdn.net/coder_pig/article/details/41075031](http://blog.csdn.net/coder_pig/article/details/41075031)

##### Fragment

Fragment是activity的界面中的一个部分或者一种行为，是在Android3.0后引入的新的API，引入的目的是为了适配大屏幕的平板。既可以把多个Fragment组合到一个Activity中来创建一个拥有多个分区的界面，也可以在多个Activity中重用一个Fragment。可以这样理解：Fragment是模块化的一段Activity，有自己的生命周期，接收自己的事件，并可以在Activity运行时被添加，替换和删除。
Fragment不能独立存在，必须嵌入到Activity这个中。

##### 核心要点

* 3.0版本都引入，即minSdk要大于11
* Fragment需要嵌套在Activity中，也可以嵌套在Fragment里，但是宿主也必须嵌套在Activity中，Fragment的生命周期受Activity的生命周期影响，不建议将Fragment嵌入Fragment里，这样会导致Fragment的生命周期不可控。
* 一定要实现onCreateView，官方说必须实现onCreate(), onCreateView(), onPause()
* Fragment的生命周期和Activity有点类似，三点比较显著的特性是Resumed:在允许中的Fragment课件  Paused:所在的Activity可见，但是得不到焦点  Stoped:所在的Activity隐藏

##### 创建一个Fragment

1. 静态添加
	1. 定义 Fragment 的布局，显示 Fragment 的内容 `fragment1.xml`
	2. 自定义一个 Fragment 类

			public class Fragmentone extends Fragment{
				public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState){
					View view = inflater.inflate(R.layout.fragment1, container, false);
					return view;
				}
			}

	3. 在需要加载 Fragment 的 Activity 对应的布局文件中添加 fragment 标签

			<fragment
				android:id="@+id/fragment1"
				android:layout_width="match_parent"
				android:layout_height="match_parent"/>

	4. Activity 的 `onCreate()` 方法调用 `setContentView()` 加载布局文件即可。

2. 动态添加 Fragment
	1. 调用 `getFragmentManager()` 获得 FragmentManager 对象 fm
	2. fm 调用 `beginTransaction()` 方法获得 Fragment 事务对象 bt
	3. bt 调用 `add()` 添加或者 `replace()` 替换 Fragment ，第一个参数是要传入的容器，第二个参数是 Fragment 对象
	4. 最后还需要调用 `bt.commit()` 提交事务，除了 `add()` 和 `replace()`方法外还有个`remove()` 移除 Fragment 的方法，同样也是需要 `commit()` 的
	
			Fragment1 f1 = new Fragment1();
			getFragmentManager().beginTransaction().replace(R.id.LinearLayout1, f1).commit()
		
##### Activity 与 Fragment 之间的交互

* 组件获取

	Fragment 获取 Activity 中的组件

	getActivity.findViewById();

	Activity 获取 Fragment 中的组件

	getFragmentManager.findFragmentById()

* 数据传递
	
	Fragment 传递数据给 Activity

	在 Fragment 中定义一个内部回调接口，然后让包含这个 Fragment 的 Activity 实现这个回调接口，Fragment 通过回调接口实现数据传递
	
	step1. 在 Fragment 中定义一个回调接口
	
		public interface CallBack{
			public void getResult(String result);
		}

	step2. 在 Fragment 中接口回调

		public getData(CallBack callBack){
			String msg = editText.getText();
			callBack.getResult(msg);
		}

	step3. 在 Activity 中使用接口回调方法读数据
	
		fragment1.getData(new CallBack(){
			@Override
			public void getResult(String result){
				Toast.makeText(getActivity(), "结果是"+result, Toast.LENGTH_SHORT).show();		
			}
		});

	#### Activity 传递数据给 Fragment
	
	在 Activity 中创建 Bundle 数据包，调用 Fragmet 对象的方法 setArguments(Bundle bundle) 将数据包传递过去，然后再 Fragment 中调用 getArguments() 获得 Bundle 对象，对 Bundle 解析之后获得

	#### Fragment 之间的数据传递

	找到接收数据的 fragment1, 然后 fragment1.setArguments() 就好了。也可以通过Activity进行间接的数据传递

### Fragment 管理与Fragment 事务

* 管理

	Activity 管理 Fragment 主要依靠 FragmentManager 可以调用 findFragmentById() 来获取指定的 fragment ,也可以通过 popBackStack() 方法弹出后台的 fragment;也可以调用addToBackStack() 将其加入栈中，使用 addOnBackStackListener() 后台栈的变化。

* 事务

	如果是增删替换 Fragment 的话，则需要借助 FragmentTransaction 对象;
	
	同时执行 Fragment 操作之后，需要 commit()

### 流程一览

* Activity 加载 Fragment 的时候

	onAttach()(挂载的意思)->onCreate()->onCreateView()->onActivityCreate()->onStart()->onResume()

* Activity 半隐藏

	onPaused()

* Activity 又获得焦点

	onResume()

* 替换 fragment 并将 fragment 加入到 Back 栈中

	onPause()->onStop()->onDestroyView()

* 按下回退键，fragment 重新显示粗来

	onCreateView()->onActivityCreate()->onStart()->onResume()

* 如果我们替换后，在commit() 之前没有将 fragment 加入 Back 栈或者退出 Activity 的话，fragment 就会被销毁

	onPause()->onStop()->onDestroyView()->onDestroy()->onDetach()(取消挂载)