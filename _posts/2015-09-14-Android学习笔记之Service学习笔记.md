---
layout: post
title:  "Android学习笔记之Service学习笔记"
date:   2015-09-15 08:34 +0800
categories: jekyll update
---
1. #### Service的生命周期

	startService方式启动：
	OnCreate --> OnStartComment --> OnDestroy

	bindService方式启动：
	OnCreate --> OnBind --> OnUnbind --> OnDestroy

	先启动后绑定方式启动Service
	OnCreate --> OnStartComment --> OnBind --> OnUnbind -- OnDestroy
	
	* ##### 如下小细节：

		* 1.1 服务先启动后绑定(顺序可换),关闭服务时unbindService和stopService 必须连续执行

		* 1.2 每次调用startService方法服务都会调用一次OnStartComment方法

		* 1.3 运行中的服务仅能被绑定一次，解绑之后不能绑定

		* 1.4 未绑定的服务解绑会抛出异常,绑定的服务解绑两次也会抛出异常

		* 1.5 记得在Service方法中清理掉不需要的资源，防止Service停止后不再使用的对象继续占用内存

		* 1.6 Service是运行在主线程中的，如果在Service中执行耗时操作一样会造成ANR,正确的耗时Service写法
		
		
				public class MyBinder extends Binder {
					public void downLoad(){
						new Thread(){
                			@Override
                			public void run() {
                	    		//do something...
                			}
            				}.start();
        				}
    				@Override
    				public void onCreate() {
        				new Thread(){
            				@Override
            				public void run() {
            					//do something..
            				}
        					}.start();
    					}
					}		
			

		* 1.7 前台Service。Service的优先级是比较低的，内容不足时系统可能会回收掉后台的Service,如果希望Service一直保持运行状态。可以使用前台Service
		
			    public void onCreate() {
			    	Intent intent = new Intent(this,MainActivity.class);
			    	PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, 0);
			    	Notification notification = new Notification.Builder(this)
                	.setTicker("有新通知")
                	.setSmallIcon(R.mipmap.ic_launcher)
                	.setContentTitle("通知标题")
                	.setContentText("通知内容")
                	.setWhen(System.currentTimeMillis())
                	.setContentIntent(pendingIntent)
                	.build();
                	startForeground(1, notification);
                	startForeground(1,notification);
                }
                
2. #### 服务与本地activity交互
	* 2.1 自定义MyBinder类继承Binder对象，MyBinder类中调用服务中的方法 如上请看1.6
	* 2.2 OnBind方法中返回MyBinder实例
			
			public IBinder onBind(Intent intent) {
              return  new MyBinder();
            }
	* 2.3 Activity中定义MyBinder对象，用于接收MyBinder实例
		
			 private MyService.MyBinder myBinder;
			
	* 2.4 定义ServiceConnection匿名类，在OnServiceConnected方法中获取自定义binder对象
	
			 public ServiceConnection conn = new ServiceConnection() {
            	@Override
            	public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
                	Log.e("TAG","onServiceConnected");
                	myBinder = (MyService.MyBinder) iBinder;
                2.5 调用MyBinder中的方法 
                	myBinder.downLoad();
            	}
            	@Override
            	public void onServiceDisconnected(ComponentName componentName) {
                	Log.e("TAG","onServiceDisconnected");
            	}
        	};
	
3. #### 调用远程Service,AIDL使用

	* 3.1 编写AIDL文件
		
			// 注意：AIDL接口中不能定义访问权限修饰符如public
			interface MyAIDLService {
            int getSum(int a,int b);
        	}
        	
        	
	* 3.2 定义MyAIDLService.Stub()匿名类并实现其中方法，在onBind方法中返回 MyAIDLService.Stub()实例
	
			public IBinder onBind(Intent intent) {
                Log.e("TAG","OnBind");
                return  stub;
            	}
            MyAIDLService.Stub stub = new MyAIDLService.Stub() {
                @Override
                public int getSum(int a, int b) throws RemoteException {
                    return a+b;
                }
            };
            
	*  3.3 因为服务端没办法通过.class的方法获取要启动的Service,所以需要隐式启动Service，在配置文件中给服务端的Service添加action
	
			<intent-filter>
    			<action android:name="ihad.cc.androidstage01.MyAIDLService"/>
    		</intent-filter>
    		
    * 3.4 新建一个工程，copyAIDL文件及其所在包名到新工程，定义一个MyAIDLService对象接收实例，一个TextVie
        			
			MyAIDLService mySerice;
			public ServiceConnection conn = new ServiceConnection() {
               @Override
               public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
                    Log.e("TAG","onServiceConnected");
                    myService = MyAIDLService.Stub.asInterface(iBinder);
                    try {
                        int sum = myService.getSum(5,4);
                        Log.e("TAG",sum+"");
                    } catch (RemoteException e) {
                        e.printStackTrace();
                    }
                }
            @Override
            public void onServiceDisconnected(ComponentName componentName) {
                        Log.e("TAG","onServiceDisconnected");
                    }
            };

	* 3.5 定义一个button通过隐式意图访问服务
		
			public void onClick(View v) {
                Intent intent = new Intent("com.example.servicetest.MyAIDLService");
                bindService(intent, connection, BIND_AUTO_CREATE);
            }