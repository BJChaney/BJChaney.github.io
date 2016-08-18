---
layout: post
title:  "Android学习笔记之Activity学习笔记"
date:   2015-10-05 18:18 +0800
categories: jekyll update
---
1. #### activity的4种状态：
	* 活动状态：activity处于前台，用户可见，可以获得焦点。 
	* 暂停状态：activity处于前台，用户可见不能获得焦点 
	* 停止状态：activity不可见，失去焦点。 
	* 销毁状态，该activity被销毁，或者该activity所在的进程被结束
	
2. #### activity的生命周期的三个循环：

	* 从开始到结束。 
	
		OnCreate–>onStart–>OnResume–>OnPause–>OnStop–>OnDestroy 
	* 2.获得焦点–>失去焦点–>再次获得焦点（当前情况下activity始终可见） 
	
		OnResume–>OnPause–>OnResume 
	* 3.创建–>不可见–>可见 
	
		OnCreate–>OnStart–>OnResume–>OnPause–>OnStop–>OnRestart–>OnStart–>OnResume
		
	** 以下两种情况下 都只会触发onPause而不会触发onStop **
	* 一个透明的包含Dialog的Activity 出现 
	* 按poweroff锁屏

3. #### 场景应用：

	* 1.用户按下Home键–>再次进入应用:
		
		属于activity生命周期中第三条循环：可以在OnStop方法中保存activity的状态，在OnRestart方法中重新回显

	* 2.当用户启动另外一个activity，再返回：
	
		OnCreate(one)–>OnStart(one)–>OnResume(one)–>OnPause(one)–>OnCreate(two)–>OnStart(two)–>OnResume(two)–>OnStop(one)//从创建到开启第二个界面 
–>OnPause(two)–>OnRestart(one)–>OnStart(one)–>OnResume(one)–>OnStop(two)–>OnDestroy(two)//从第二个界面返回

	* 3.当用户从横屏切换到竖屏，再切换回来 
	
		默认情况下，系统会先销毁之前创建的activity后再重新创建
		
		OnCreate–>OnStart–>OnResume–>OnPause–>OnSaveInstanceState–>OnStop–>OnDestroy–>OnCreate–>OnStart–>OnRestoreInstanceState–>OnResume//activity切换横屏 
–>OnPause–>OnSaveInstanceState–>OnStop–>OnDestroy–>OnCreate–>OnStart–>OnRestoreInstanceState–>OnResume//重新切换到竖屏 

	**注： **
	
	谷歌提供了OnSaveInstanceState(保存实例状态)方法和OnRestoreInstanceState(恢复实例状态)方法，方法接收一个Bundle对象,仅横竖屏切换时会调用OnRestoreInstanceState方法 
	
	使用方法类似Map集合，保存信息为键值对的形式，可以保存基本类型，数组和ArrayList集合,如果想传输对象，可以使用putSerializable或者putParcelable 
	
	要求该对象必须实现Serializable或者Parcelable接口，通常推荐使用Parcelable，Serializable内部是通过反射来实现的比较消耗性能，速度远不如Parcelabl，不同Activity之间也可以通过这种方式进行传值。 

	**Parcelable接口实现：**
	
		public class Person implements Parcelable{
			public String name;
			public static final Creator<Person> creator = new Creator<Person>() {
            	@Override
            	public Person createFromParcel(Parcel parcel){
            		Person person = new Person();
                	person.name = parcel.readString();
                	return person;
                }
                @Override
                public Person[] newArray(int i) {
                	return new Person[i];
                }
            };
            @Override
            public int describeContents() {
            	return 0;
            }
            @Override
            public void writeToParcel(Parcel parcel, int i) {
            	parcel.writeString(name);
            }
         }
         
	
	如果想Activity屏幕切换不被销毁重新创建，可以在配置文件中添加如下配置项 
	而Adnroid 3.2以后的SDK必须添加一个screenSize属性，具体如下 
	android:configChanges=”orientation|screenSize” 
	
	**注：仅测试过Genymotion模拟器**
	
4. #### Activity启动模式：
	Activity启动模式设置：
		
		<activity android:name=".MainActivity" android:launchMode="standard" />
		
	Activity的四种启动模式：
	
	1. standard 模式启动模式（默认），每次激活Activity时都会创建Activity，并放入任务栈中。

	2. singleTop 如果在任务的栈顶正好存在该Activity的实例， 就重用该实例，否者就会创建新的实例并放入栈顶(即使栈中已经存在该Activity实例，只要不在栈顶，都会创建实例)。

	3. singleTask 如果在栈中已经有该Activity的实例，就重用该实例(会调用实例的onNewIntent())。重用时，会让该实例回到栈顶，因此在它上面的实例将会被移除栈。如果栈中不存在该实例，将会创建新的实例放入栈中。

	4. singleInstance 在一个新栈中创建该Activity实例，并让多个应用共享改栈中的该Activity实例。一旦改模式的Activity的实例存在于某个栈中，任何应用再激活改Activity时都会重用该栈中的实例，其效果相当于多个应用程序共享一个应用，不管谁激活该Activity都会进入同一个应用中。

