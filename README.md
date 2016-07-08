# HandlerTest
multify processes communication

1. Broadcase

2. aidl

   a. 定义aidl,一定是接口
  b. 接口中出了基本类型,不会自动import其它类型. (此是坑点,会导致.java无法自动生成)
 c.  UI Process 呼叫 Remote Service
    通过BindService, 传入ServiceConnection,在ServiceConnection中的回调函数中得到Remote的Binder
  例如:
public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
    IService = IRemoteService.Stub.asInterface(iBinder);
    Log.d("tst", "service binded...");
}

serverCon = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
        IService = IRemoteService.Stub.asInterface(iBinder);
        Log.d("tst", "service binded...");
    }

    @Override
    public void onServiceDisconnected(ComponentName componentName) {
        Log.d("tst", "service unbinded...");
    }
};
Intent bindIntent = new Intent(this, BackgroundService.class);
bindService(bindIntent, serverCon, BIND_AUTO_CREATE);


d. Remote Service 呼叫 UI Process
    通过c步骤得到Remote Service的binder后，直接将UI Process的Binder引用传给Remote Service Binder
如下Code，将UI Process的binder引用传到 Remote Service binder中，来提供回调路径。
mCb = new RemoteCallback();
try {
    IService.registerCallback(mCb);
} catch (RemoteException e) {
    e.printStackTrace();
}


  IRemoteCallback.aidl
interface IRemoteCallback {
     void onDataUpdate();
}
IRemoteCallback.aidl
interface IRemoteService {
    int plus(int a, int b);
    int sub(int a, int b);
    void registerCallback(IRemoteCallback cb);
    void unRegisterCallback(IRemoteCallback cb);
}
private class LocalBinder extends IRemoteService.Stub{


    @Override
    public int plus(int a, int b) throws RemoteException {
        return a + b;
    }

    @Override
    public int sub(int a, int b) throws RemoteException {
        return a - b;
    }

    @Override
    public void registerCallback(IRemoteCallback cb) throws RemoteException {
        mCallback = cb;
    }

    @Override
    public void unRegisterCallback(IRemoteCallback cb) throws RemoteException {
        mCallback = null;
    }
}
private  class RemoteCallback extends IRemoteCallback.Stub
{

    @Override
    public void onDataUpdate() throws RemoteException {
        Log.d("tst", " UI Process... onDataUpdate..." );
    }
}

