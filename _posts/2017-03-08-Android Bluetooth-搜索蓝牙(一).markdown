---
layout: post
title: Android Bluetooth-搜索蓝牙(一)
date: 2017-03-08 16:00:00.000000000 +09:00
---

前言
>Hello，现在的工作，必须要学习Bluetooth的一些使用,在这里记录一下

一.蓝牙权限
不管如何,开发蓝牙,首先加上权限再说
```
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
```
二.蓝牙API
1.BluetoothAdapter   
本地蓝牙适配器(如:自己的手机)

2.BlueDevice           
 远程蓝牙(如:一个蓝牙音箱)

3.BluetoothSocket   
蓝牙socket的接口

4.BluetoothServerSocket
开放的服务器socket，它监听接受的请求（与TCP ServerSocket类似）

------------------
三.使用
***1.BluetoothAdapter下的方法***
//开始搜索
startDiscovery()
//取消搜索
cancelDiscovery() 
//直接打开蓝牙
enable()
//直接关闭蓝牙
disable()
//弹窗申请权限打开蓝牙
Intemtenabler=new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
startActivityForResult(enabler,reCode);
//获得本地蓝牙名字
getName()
//本地蓝牙地址
getAddress()
//获取本地蓝牙适配器
getDefaultAdapter()
//根据蓝牙地址获取远程蓝牙设备
getRemoteDevice(String address)
//获取本地蓝牙适配器当前状态
getState()
//判断当前是否正在查找设备
isDiscovering()
//判断蓝牙是否打开
isEnabled()
//根据名称，UUID创建并返回BluetoothServerSocket
listenUsingRfcommWithServiceRecord(String name,UUID uuid)

--------------------
***2.实际使用***
```
// 获取本地蓝牙适配器
BluetoothAdapter mBluetoothAdapter = BluetoothAdapter.getDefaultAdapter();
//获得本地蓝牙name和mac地址
mBluetoothAdapter.getName();
mBluetoothAdapter.getAddress();
```
```
 // 判断有没有蓝牙硬件支持
if (mBluetoothAdapter == null) {   
 Toast.makeText(this, "设备不支持蓝牙", Toast.LENGTH_SHORT).show();    finish();
 }
```
```
 // 判断是否打开了蓝牙
        if (!mBluetoothAdapter.isEnabled()) {
            // 我们通过startActivityForResult()方法发起的Intent将会在onActivityResult()回调方法中获取用户的选择，比如用户单击了Yes开启，
            // 那么将会收到RESULT_OK的结果，
            // 如果RESULT_CANCELED则代表用户不愿意开启蓝牙
            // 弹出对话框提示用户是否打开
            Intent intent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
            startActivityForResult(intent, 1);
            // 没有提示强制打开蓝牙(用enable()方法来开启，无需询问用户(实惠无声息的开启蓝牙设备),这时就需要用到android.permission.BLUETOOTH_ADMIN权限)
            // mBluetoothAdapter.enable();
```
```
// 获取已经配对的设备
    Set<BluetoothDevice>  myDevices = mBluetoothAdapter.getBondedDevices();
        // 判断是否有配对过的设备
        if (myDevices.size() > 0) {
            for (BluetoothDevice device :  myDevices ) {
                // 遍历到列表中
                tv_devices.append(device.getName() + ":" + device.getAddress());
                Log.i("已配对设备", tvDevices.getText().toString());
            }
        }

// 获得连接中蓝牙的name和mac地址
    BluetoothDevice device=intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE);		
    String bltname= device.getName().toString();
    String bltmac= device.getAddress().toString();
```
----------------
**xml代码**
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.example.zxnet.mybluetoothdome.MainActivity">

    <Button
        android:id="@+id/btn_search"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="搜索蓝牙设备"
        android:background="#54FF9F"
        />

    <ListView
        android:id="@+id/mylistview"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/btn_search"
        android:textSize="18sp"
        />
</RelativeLayout>

```
**MainActivity.class类**
```
import android.Manifest;
import android.bluetooth.BluetoothAdapter;
import android.bluetooth.BluetoothDevice;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;
import android.content.pm.PackageManager;
import android.os.Build;
import android.os.Bundle;
import android.support.annotation.NonNull;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.ListView;
import android.widget.Toast;

import java.util.ArrayList;
import java.util.List;
import java.util.Set;

public class MainActivity extends AppCompatActivity implements AdapterView.OnItemClickListener {
    private static final String TAG = "MainActivity";
    // 本地蓝牙适配器
    private BluetoothAdapter mBluetoothAdapter;
    // 搜索按钮
    private Button btn_search;
    //搜索到蓝牙显示列表
    private ListView mylistview;
    // listview的adapter
    private ArrayAdapter<String> mylistAdapter;
    // 存储搜索到的蓝牙
    private List<String> bluetoothdeviceslist = new ArrayList<String>();

    private static final int PERMISSION_REQUEST_COARSE_LOCATION = 1;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        checkBluetoothPermission();
        initview();
        SearchBluetoot();

    }

    private void initview() {
        mylistview = (ListView) findViewById(R.id.mylistview);
        btn_search = (Button) findViewById(R.id.btn_search);

        // 获取本地蓝牙适配器
        mBluetoothAdapter = BluetoothAdapter.getDefaultAdapter();

        btn_search.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                setTitle("正在搜索...");
                // 判断是否在搜索,如果在搜索，就取消搜索
                if (mBluetoothAdapter.isDiscovering()) {
                    mBluetoothAdapter.cancelDiscovery();
                }
                // 开始搜索
                mBluetoothAdapter.startDiscovery();
            }
        });

    }

    public void SearchBluetoot() {
        // 判断有没有蓝牙硬件支持
        if (mBluetoothAdapter == null) {
            Toast.makeText(this, "设备不支持蓝牙", Toast.LENGTH_SHORT).show();
            finish();
        }

        // 判断是否打开了蓝牙
        if (!mBluetoothAdapter.isEnabled()) {
            // 我们通过startActivityForResult()方法发起的Intent将会在onActivityResult()回调方法中获取用户的选择，比如用户单击了Yes开启，
            // 那么将会收到RESULT_OK的结果，
            // 如果RESULT_CANCELED则代表用户不愿意开启蓝牙
            // 弹出对话框提示用户是否打开
            Intent intent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
            startActivityForResult(intent, 1);
            // 没有提示强制打开蓝牙(用enable()方法来开启，无需询问用户(实惠无声息的开启蓝牙设备),这时就需要用到android.permission.BLUETOOTH_ADMIN权限)
            // mBluetoothAdapter.enable();

            // 获的已经配对的设备
            Set<BluetoothDevice> myDevices = mBluetoothAdapter.getBondedDevices();
            /// 判断是否有配对过的设备
            if (myDevices.size() > 0) {
                for (BluetoothDevice device : myDevices) {
                    // 遍历到列表中
                    bluetoothdeviceslist.add(device.getName() + ":" + device.getAddress() + "\n");
                }
            }
// adapter
            mylistAdapter = new ArrayAdapter<String>(this,android.R.layout.simple_list_item_1, android.R.id.text1, bluetoothdeviceslist);
            mylistview.setAdapter(mylistAdapter);

            // 找到设备的广播
            IntentFilter filter = new IntentFilter(BluetoothDevice.ACTION_FOUND);
            // 注册广播
            registerReceiver(myreceiver, filter);
            // 搜索完成的广播
            filter = new IntentFilter(BluetoothAdapter.ACTION_DISCOVERY_FINISHED);
            // 注册广播
            registerReceiver(myreceiver, filter);
        }
    }

    // 广播接收器
    private final BroadcastReceiver myreceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            // 收到的广播类型
            String action = intent.getAction();
            // 发现设备的广播
            if (BluetoothDevice.ACTION_FOUND.equals(action)) {
                // 从intent中获取设备
                BluetoothDevice device = intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE);
                // 判断是否配对过
                if (device.getBondState() != BluetoothDevice.BOND_BONDED) {
                    // 添加到列表
                    bluetoothdeviceslist.add(device.getName() + ":"+ device.getAddress() + "\n");
                    mylistAdapter.notifyDataSetChanged();
                }
                // 搜索完成
            } else if (BluetoothAdapter.ACTION_DISCOVERY_FINISHED.equals(action)) {
                setTitle("搜索完成！");
            }
        }
    };

    /*
           校验蓝牙权限
          */
    private void checkBluetoothPermission() {
        if (Build.VERSION.SDK_INT >=Build.VERSION_CODES.M) {
            // Android M Permission check
            if (checkSelfPermission(Manifest.permission.ACCESS_COARSE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
                requestPermissions(new String[]{Manifest.permission.ACCESS_COARSE_LOCATION},PERMISSION_REQUEST_COARSE_LOCATION);
            }
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        switch (requestCode) {
            case PERMISSION_REQUEST_COARSE_LOCATION:
                if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {

                }
                break;
        }
    }

    @Override
    public void onItemClick(AdapterView<?> parent, View view, int position, long id) {

    }
}

```
------------------------------------
以上就是蓝牙基础的使用,也比较简单,要开发蓝牙,这些是必须了解的,要想了解更多,请仔细看官方的api开发文档,如果对你有帮助,请点个赞吧!!


EDN