# android-serialport-api

Android上使用串口的一个Demo，其中也提供了JNI部分的源代码和so文件，可以自己另外编译，也可以直接使用。

## 修复google 提供的串口工具android-serialport-api丢失字节bug。

* 由于在issues多人提出丢失字节的bug，却没人修改，所以程梦真修复该bug；
* 其真实的bug原因是由于关闭Activity的时候输入流的read方法被阻塞，以至于关闭Activity，但GC不能释放，导致下一次接收导数据时，数据先给了没有被释放的Activity，然后之后收到的数据才给新创建的Activity；
* 在SerialPortActivity中ReadThread的完整代码为：

```java
    private class ReadThread extends Thread {
        @Override
        public void run() {
            super.run();
            while(!isInterrupted()) {
                int size;
                try {
                    byte[] buffer = new byte[64];
                    if (mInputStream == null) return;
                    if (mInputStream.available() > 0) {
                        size = mInputStream.read(buffer);
                        if (size > 0) {
                            onDataReceived(buffer, size);
                        }
                    }
                    SystemClock.sleep(100);
                } catch (IOException e) {
                    e.printStackTrace();
                    return;
                }
            }
        }
    }
```
