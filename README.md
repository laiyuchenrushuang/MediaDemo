# VehicleSaleService
说多了没用直接上代码======

# LoginByIdcardActivity.java

``package com.seatrend.cd.vehiclesaleservice.activity;

import android.Manifest;
import android.app.Dialog;
import android.content.ComponentName;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.os.Build;
import android.os.Bundle;
import android.os.Handler;
import android.support.annotation.NonNull;
import android.support.annotation.RequiresApi;
import android.support.v4.app.ActivityCompat;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.LinearLayout;
import android.widget.ProgressBar;
import android.widget.TextView;
import android.widget.Toast;

import com.seatrend.cd.vehiclesaleservice.R;
import com.seatrend.cd.vehiclesaleservice.common.BaseActivity;
import com.seatrend.cd.vehiclesaleservice.common.Constants;
import com.seatrend.cd.vehiclesaleservice.entity.CheckAppVersion;
import com.seatrend.cd.vehiclesaleservice.entity.CommonProgress;
import com.seatrend.cd.vehiclesaleservice.entity.CommonResponse;
import com.seatrend.cd.vehiclesaleservice.entity.LoginEntity;
import com.seatrend.cd.vehiclesaleservice.entity.User;
import com.seatrend.cd.vehiclesaleservice.persenter.LoginPersenter;
import com.seatrend.cd.vehiclesaleservice.util.AppUtils;
import com.seatrend.cd.vehiclesaleservice.util.GsonUtils;
import com.seatrend.cd.vehiclesaleservice.util.LoadingDialog;
import com.seatrend.cd.vehiclesaleservice.util.OtherUtils;
import com.seatrend.cd.vehiclesaleservice.util.ProgressDialog;
import com.seatrend.cd.vehiclesaleservice.view.LoginView;

import java.io.File;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import butterknife.BindView;
import butterknife.ButterKnife;
import butterknife.OnClick;

public class LoginByIdcardActivity extends BaseActivity implements LoginView {


    @BindView(R.id.tv_user)
    TextView tvUser;
    @BindView(R.id.ll_user)
    LinearLayout llUser;
    @BindView(R.id.btn_dqsfz)
    Button btnDqsfz;
    @BindView(R.id.tv_app_version)
    TextView tvAppVersion;



    private int permissionCount = 0;
    private LoginPersenter mLoginPersenter;
    private Dialog progressdialog;
    private ProgressBar progressBar;
    private TextView tvPro;
    private String apkPath;
    private String appdownloadurl;

    private static final int NFC_READ_IDCARD=1000;
    private static final int RLSB_READ_IDCARD=1001;

    public byte[] userIdCardPhoto;

    @Override
    public int getLayout() {
        return R.layout.activity_login_by_idcard;
    }

    @Override
    public void initView() {
        ivBack.setVisibility(View.GONE);
        mLoginPersenter = new LoginPersenter(this);
        setPageTitle("身份证登录");
        tvAppVersion.setText(getString(R.string.cur_version,AppUtils.getVersionName(this),Constants.UPDATA_TIME));
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            AppRequestPermissions();
        }
        checkAppVersion();
    }

    private void doLogin(String cardId) {


        Map<String, String> map = new HashMap<>();
        map.put("username", cardId);
        map.put("password", "123456");
        LoadingDialog.getInstance().showLoadDialog(this);
        mLoginPersenter.doNetworkTask(map, Constants.APP_LOGIN);
    }

    @RequiresApi(api = Build.VERSION_CODES.M)
    private void AppRequestPermissions() {

        // String [] permission={};
        final List<String> permission = new ArrayList<>();
        if (checkSelfPermission(Manifest.permission.READ_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
            permission.add(Manifest.permission.READ_EXTERNAL_STORAGE);
        }
        if (checkSelfPermission(Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
            permission.add(Manifest.permission.CAMERA);
        }

        if (checkSelfPermission(Manifest.permission.READ_PHONE_STATE) != PackageManager.PERMISSION_GRANTED) {
            permission.add(Manifest.permission.READ_PHONE_STATE);
        }
        if (permission.size() > 0) {
            permissionCount = permission.size();

            ActivityCompat.requestPermissions(this, permission.toArray(new String[permission.size()]), 1);

        }


    }

    private void downloadApk() {
        File file = new File(Constants.FILE_PATH);
        if (!file.exists()) {
            file.mkdirs();
        }
        File f = new File(file, "vehiclesaleservice.apk");
        apkPath = f.getPath();
        mLoginPersenter.downloadFile(appdownloadurl, f);
    }

    @RequiresApi(api = Build.VERSION_CODES.M)
    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if (requestCode == 1) {

            if (checkSelfPermission(Manifest.permission.READ_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
                showToast("您有相关权限未同意，无法使用该软件");
                finish();
            }
            if (checkSelfPermission(Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
                showToast("您有相关权限未同意，无法使用该软件");
                finish();
            }

            if (checkSelfPermission(Manifest.permission.READ_PHONE_STATE) != PackageManager.PERMISSION_GRANTED) {
                showToast("您有相关权限未同意，无法使用该软件");
                finish();
            }
        }
    }


    @Override
    public void netWorkTaskSuccess(CommonResponse commonResponse) {
        LoadingDialog.getInstance().dismissLoadDialog();

        if (commonResponse == null) {
            return;
        }
        Log.i("sdsss","---  "+commonResponse.getResponseString());
        try {

            if (commonResponse.getUrl().equals(Constants.APP_LOGIN)) {
                LoginEntity loginEntity = GsonUtils.gson(commonResponse.getResponseString(), LoginEntity.class);
                if (loginEntity != null && loginEntity.getData() != null) {
                    LoginEntity.DataBean data = loginEntity.getData();
                    User.LXR = data.getLxr();
                    User.SSQY = data.getSsqy();
                    User.LXDH = data.getLxdh();
                    User.YXDZ = data.getYxdz();
                    User.TOKEN = data.getJwtToken();
                    User.USER_NAME = data.getUsername();
                    User.XSDMC = data.getXsdmc();
                    User.XSDDD = data.getXsddd();
                    if (data.getSeaDepartment() != null) {
                        User.XSDZWMC = data.getSeaDepartment().getBmmc();
                    }
                    startActivity(new Intent(this, MainActivity.class));
                    finish();
                } else {
                    showToast("未获取到用户信息");
                }
            } else if (commonResponse.getUrl().equals(Constants.CHECK_VERSION)) {
                CheckAppVersion checkAppVersion = GsonUtils.gson(commonResponse.getResponseString(), CheckAppVersion.class);
                CheckAppVersion.DataBean data = checkAppVersion.getData();
                appdownloadurl = data.getAppdownloadurl();
                if (!AppUtils.getVersionName(this).equals(data.getAppversion())) {
                    showTipsDialog(data.getUpdatemessage());
                }

            }
        } catch (Exception e) {
            showToast(e.getMessage());

        }
    }

    private void checkAppVersion() {
        mLoginPersenter.doNetworkTask(new HashMap<String, String>(), Constants.CHECK_VERSION);
    }

    private void showTipsDialog(String mes) {
        final Dialog mDialog = new Dialog(this);
        mDialog.setContentView(R.layout.dialog_tips);
        mDialog.setCanceledOnTouchOutside(false);
        TextView tvMsg = mDialog.findViewById(R.id.tv_tips_msg);
        Button btnOk = mDialog.findViewById(R.id.btn_ok);
        Button btnCancel = mDialog.findViewById(R.id.btn_cancel);
        tvMsg.setText(getString(R.string.update_message, mes));
        tvMsg.setLineSpacing(6, 1);
        btnOk.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mDialog.dismiss();
                downloadApk();
            }
        });
        btnCancel.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mDialog.dismiss();

            }
        });
        mDialog.show();
    }

    @Override
    public void netWorkTaskfailed(CommonResponse commonResponse) {
        LoadingDialog.getInstance().dismissLoadDialog();
        Log.i("sdsss","---  "+commonResponse.getResponseString());
        if (!Constants.CHECK_VERSION.equals(commonResponse.getUrl())) {
            showErrorDialog(commonResponse.getResponseString());
        }

    }

    @Override
    public void downloadProgress(CommonProgress commonProgress) {
        //Log.i("downloadProgress"," ---  "+commonProgress.getProgress());
        if (progressdialog == null) {
            showProgressDialog();
        } else {
            if (!progressdialog.isShowing()) {
                progressdialog.show();
            }
        }
        if (progressdialog != null && progressdialog.isShowing()) {
            progressBar.setProgress((int) Double.parseDouble(commonProgress.getProgress()));
            tvPro.setText(String.format("%s%%", commonProgress.getProgress()));
        }
        if ("100.0".equals(commonProgress.getProgress()) && progressdialog != null && progressdialog.isShowing()) {
            progressdialog.dismiss();
            installApk();
        }

    }

    private void installApk() {
        AppUtils.installApp(this, apkPath);
    }

    private void showProgressDialog() {
        progressdialog = ProgressDialog.getProgressDialog(this);
        progressBar = progressdialog.findViewById(R.id.pb_pro);
        tvPro = progressdialog.findViewById(R.id.tv_pro);
        TextView tvMsg = progressdialog.findViewById(R.id.tv_msg);
        tvMsg.setText(getString(R.string.downloading));
        progressdialog.show();
    }



    @OnClick({R.id.btn_dqsfz, R.id.btn_setting})
    public void onViewClicked(View view) {
        switch (view.getId()) {
            case R.id.btn_dqsfz:
//                goNfcReadPlugin();
                doLogin("511023198509290031");
                break;

            case R.id.btn_setting:
                startActivity(new Intent(this,SettingActivity.class));
                break;
        }
    }

    private void goNfcReadPlugin() {
        try {
            Intent intent = new Intent(Intent.ACTION_VIEW);
            intent.setComponent(new ComponentName("com.seatrend.cd.nfcread", "com.seatrend.cd.nfcread.IdCardReadActivity"));
            startActivityForResult(intent, NFC_READ_IDCARD);
        } catch (Exception e) {
            Toast.makeText(this, "未找到NFC身份证读取插件，请先安装插件", Toast.LENGTH_SHORT).show();

        }

    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == NFC_READ_IDCARD && resultCode == RESULT_OK && data != null) {
           String name= data.getStringExtra(Constants.NAME);
           String number= data.getStringExtra(Constants.NUMBER);
           userIdCardPhoto= data.getByteArrayExtra(Constants.PHOTO);
           tvUser.setText(number);
           llUser.setVisibility(View.VISIBLE);

            if (userIdCardPhoto!=null){
                OtherUtils.goFaceComparePlugin(this, userIdCardPhoto, RLSB_READ_IDCARD);
            }
        }else if (requestCode == RLSB_READ_IDCARD && resultCode == RESULT_OK && data != null){
           // String path = data.getStringExtra(Constants.PATH);
            new Handler().postDelayed(new Runnable() {
                @Override
                public void run() {
                    doLogin(tvUser.getText().toString());
                }
            },1000);

        }

    }
}



# 下载进度的HttpService.java


package com.seatrend.cd.vehiclesaleservice.http;

import android.annotation.SuppressLint;
import android.os.Handler;
import android.os.Message;
import android.system.Os;
import android.text.TextUtils;
import android.util.Log;

import com.google.gson.JsonSyntaxException;
import com.seatrend.cd.vehiclesaleservice.common.BaseModule;
import com.seatrend.cd.vehiclesaleservice.common.Constants;
import com.seatrend.cd.vehiclesaleservice.common.MyApplication;
import com.seatrend.cd.vehiclesaleservice.entity.BaseEntity;
import com.seatrend.cd.vehiclesaleservice.entity.CommonProgress;
import com.seatrend.cd.vehiclesaleservice.entity.CommonResponse;
import com.seatrend.cd.vehiclesaleservice.entity.ErrorEntity;
import com.seatrend.cd.vehiclesaleservice.entity.User;
import com.seatrend.cd.vehiclesaleservice.module.ProgressModule;
import com.seatrend.cd.vehiclesaleservice.util.GsonUtils;
import com.seatrend.cd.vehiclesaleservice.util.NetUtils;
import com.seatrend.cd.vehiclesaleservice.util.SharedPreferencesUtils;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.URL;
import java.net.URLConnection;
import java.text.DecimalFormat;
import java.util.List;
import java.util.Map;
import java.util.concurrent.TimeUnit;

import okhttp3.Call;
import okhttp3.Callback;
import okhttp3.FormBody;
import okhttp3.MediaType;
import okhttp3.MultipartBody;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.RequestBody;
import okhttp3.Response;
import okhttp3.ResponseBody;

/**
 * Created by seatrend on 2018/8/20.
 */

public class HttpService {

    private static HttpService mHttpService;
    private BaseModule mBaseModule;

    private final  int SUCCESS_CODE=0;
    private final  int FAILED_CODE=1;
    private final  int PREGRESS_CODE=2;

    private final int TIME_OUT=60*1000;


    private OkHttpClient mOkHttpClient = new OkHttpClient.Builder()
            .readTimeout(TIME_OUT, TimeUnit.MILLISECONDS)
            .writeTimeout(TIME_OUT, TimeUnit.MILLISECONDS)
            .connectTimeout(20*1000, TimeUnit.MILLISECONDS)
            .build();


    public static HttpService getInstance() {
        if (mHttpService == null) {
            synchronized (HttpService.class) {
                if (mHttpService == null) {
                    mHttpService = new HttpService();
                }
            }
        }
        return mHttpService;
    }

    @SuppressLint("HandlerLeak")
    private Handler mHandler=new Handler(){
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);

            switch (msg.what){
                case SUCCESS_CODE:
                    CommonResponse commonResponse1 = (CommonResponse) msg.obj;
                    mBaseModule.doWorkResults(commonResponse1,true);
                    break;

                case FAILED_CODE:
                    CommonResponse commonResponse2 = (CommonResponse) msg.obj;
                    mBaseModule.doWorkResults(commonResponse2,false);
                    break;
                case PREGRESS_CODE:
                    CommonProgress commonProgress = (CommonProgress) msg.obj;
                    ((ProgressModule)mBaseModule).downloadProgress(commonProgress);
                    break;
            }
        }
    };

//Request request = addHeaders().url(requestUrl).build();
    public void getDataFromServer(Map<String, String> map, final String url, String method, BaseModule module) {
        this.mBaseModule=module;
        if(!NetUtils.isNetworkAvailable(MyApplication.getMyApplicationContext())){
            Message message = Message.obtain();
            message.what=FAILED_CODE;
            CommonResponse commonResponse=new CommonResponse();
            commonResponse.setUrl(url);
            commonResponse.setResponseString("网络异常，请检查网络是否连接");
            message.obj=commonResponse;
            mHandler.sendMessage(message);
            return;
        }
        String baseUrl = "http://"+ SharedPreferencesUtils.getIpAddress() + ":" + SharedPreferencesUtils.getPort();
        final String finalUrl = baseUrl+url;
        Request request=null;
        try {

            if(method.equals(Constants.GET)){
                StringBuffer buffer=new StringBuffer();
                buffer.append("?");
                for (Map.Entry<String, String> entry : map.entrySet()) {
                    buffer.append(entry.getKey().trim()+"="+entry.getValue().trim()+"&");
                }
                String s = buffer.toString();
                String parameter=s.substring(0,s.length()-1);
                if(url.equals(Constants.GET_CODE) || url.equals(Constants.CHECK_IDCARD) || url.equals(Constants.CHECK_VERSION)){
                    request = new Request.Builder()
                            .url(finalUrl+parameter)
                            .get()
                            .build();
                }else {
                    request = new Request.Builder()
                            .url(finalUrl+parameter)
                            .get()
                            .addHeader(Constants.QAUTH,User.TOKEN)
                            .build();
                }

                Log.i("HttpService",finalUrl+parameter);
            }else {

                FormBody.Builder builder = new FormBody.Builder();
                for (Map.Entry<String, String> entry : map.entrySet()) {
                    builder.add(entry.getKey().trim(), entry.getValue().trim());

                }

                RequestBody requestBody = builder.build();

                request = new Request.Builder()
                        .url(finalUrl)
                        .post(requestBody)
                        .addHeader(Constants.QAUTH,User.TOKEN)
                        .build();
                Log.i("HttpService",finalUrl);
            }
        }catch (Exception e){
            Message message = Message.obtain();
            message.what=FAILED_CODE;
            CommonResponse commonResponse=new CommonResponse();
            commonResponse.setUrl(url);
            commonResponse.setResponseString(e.getMessage());
            message.obj=commonResponse;
            mHandler.sendMessage(message);
            return;
        }



        mOkHttpClient.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                Message message = Message.obtain();
                message.what=FAILED_CODE;
                CommonResponse commonResponse=new CommonResponse();
                commonResponse.setUrl(url);
                commonResponse.setResponseString(e.getMessage());
                message.obj=commonResponse;
                mHandler.sendMessage(message);
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                Message message = Message.obtain();
                String resp = response.body().string();
                if(TextUtils.isEmpty(resp)){
                    message.what=FAILED_CODE;
                    CommonResponse commonResponse=new CommonResponse();
                    commonResponse.setUrl(url);
                    commonResponse.setResponseString("服务器响应内容为空");
                    message.obj=commonResponse;
                    mHandler.sendMessage(message);
                    return;
                }
                try {
                        BaseEntity baseEntity = GsonUtils.gson(resp, BaseEntity.class);
                        //虽然响应成功，有可能数据不对
                        boolean status = baseEntity.isStatus();
                        int code = baseEntity.getCode();
                        if(status && code == 0){
                            message.what=SUCCESS_CODE;
                        }else {
                            message.what=FAILED_CODE;
                        }
                        CommonResponse commonResponse=new CommonResponse();
                        commonResponse.setUrl(url);
                        commonResponse.setResponseString(resp);
                        message.obj=commonResponse;


                } catch (JsonSyntaxException e){
                    try{
                        ErrorEntity errorEntity = GsonUtils.gson(resp, ErrorEntity.class);
                        message.what=FAILED_CODE;
                        CommonResponse commonResponse=new CommonResponse();
                        commonResponse.setUrl(url);
                        commonResponse.setResponseString("JsonSyntaxException "+errorEntity.toString());
                        message.obj=commonResponse;
                    }catch (JsonSyntaxException e1){
                        message.what=FAILED_CODE;
                        CommonResponse commonResponse=new CommonResponse();
                        commonResponse.setUrl(url);
                        commonResponse.setResponseString(resp);
                        message.obj=commonResponse;
                    }
                }
                mHandler.sendMessage(message);
            }
        });

    }

    public void getDataFromServerByJson(String json, final String url, BaseModule module) {
        this.mBaseModule=module;
        String baseUrl = "http://"+SharedPreferencesUtils.getIpAddress() + ":" + SharedPreferencesUtils.getPort();
        final String finalUrl = baseUrl+url;

        Request request;
        try {
                MediaType mjson = MediaType.parse("application/json; charset=utf-8");
                RequestBody  requestBody= RequestBody.create(mjson,json);

                  request = new Request.Builder()
                        .url(finalUrl)
                        .post(requestBody)
                        .build();
                Log.i("HttpService",finalUrl);
        }catch (Exception e){
            Message message = Message.obtain();
            message.what=FAILED_CODE;
            CommonResponse commonResponse=new CommonResponse();
            commonResponse.setUrl(url);
            commonResponse.setResponseString(e.getMessage());
            message.obj=commonResponse;
            mHandler.sendMessage(message);
            return;
        }



        mOkHttpClient.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                Message message = Message.obtain();
                message.what=FAILED_CODE;
                CommonResponse commonResponse=new CommonResponse();
                commonResponse.setUrl(url);
                commonResponse.setResponseString(e.getMessage());
                message.obj=commonResponse;
                mHandler.sendMessage(message);
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                Message message = Message.obtain();
                String resp = response.body().string();
                if(TextUtils.isEmpty(resp)){
                    message.what=FAILED_CODE;
                    CommonResponse commonResponse=new CommonResponse();
                    commonResponse.setUrl(url);
                    commonResponse.setResponseString("服务器响应内容为空");
                    message.obj=commonResponse;
                    mHandler.sendMessage(message);
                    return;
                }
                try {
                    BaseEntity baseEntity = GsonUtils.gson(resp, BaseEntity.class);
                    //虽然响应成功，有可能数据不对
                    boolean status = baseEntity.isStatus();
                    int code = baseEntity.getCode();
                    if(status && code == 0){
                        message.what=SUCCESS_CODE;
                    }else {
                        message.what=FAILED_CODE;
                    }
                    CommonResponse commonResponse=new CommonResponse();
                    commonResponse.setUrl(url);
                    commonResponse.setResponseString(resp);
                    message.obj=commonResponse;
                } catch (JsonSyntaxException e){
                    try{
                        ErrorEntity errorEntity = GsonUtils.gson(resp, ErrorEntity.class);
                        message.what=FAILED_CODE;
                        CommonResponse commonResponse=new CommonResponse();
                        commonResponse.setUrl(url);
                        commonResponse.setResponseString("JsonSyntaxException "+errorEntity.toString());
                        message.obj=commonResponse;
                    }catch (JsonSyntaxException e1){



                        message.what=FAILED_CODE;
                        CommonResponse commonResponse=new CommonResponse();
                        commonResponse.setUrl(url);

                        commonResponse.setResponseString("JsonSyntaxException"+e1.getMessage());
                        message.obj=commonResponse;
                    }
                }
                mHandler.sendMessage(message);
            }
        });

    }
    public void downLoadFileFromServer(Map<String, String> map, final File file, final String url, String method, BaseModule module) {
        this.mBaseModule=module;
        String baseUrl = "http://"+SharedPreferencesUtils.getIpAddress() + ":" + SharedPreferencesUtils.getPort();
        final String finalUrl = baseUrl+url;
        Request request=null;
        try {
            if(method.equals(Constants.GET)){
                StringBuffer buffer=new StringBuffer();
                buffer.append("?");
                for (Map.Entry<String, String> entry : map.entrySet()) {
                    buffer.append(entry.getKey().trim()+"="+entry.getValue().trim()+"&");
                }
                String s = buffer.toString();
                String parameter=s.substring(0,s.length()-1);
                request = new Request.Builder()
                        .url(finalUrl+parameter)
                        .get()
                        .addHeader(Constants.QAUTH,User.TOKEN)
                        .build();
                Log.i("HttpService",finalUrl+parameter);
            }else {
                FormBody.Builder builder = new FormBody.Builder();
                for (Map.Entry<String, String> entry : map.entrySet()) {
                    builder.add(entry.getKey().toString(), entry.getValue().trim());
                }
                request = new Request.Builder()
                        .url(finalUrl)
                        .addHeader(Constants.QAUTH,User.TOKEN)
                        .post(builder.build())
                        .build();
            }
        }catch (Exception e){
            Message message = Message.obtain();
            message.what=FAILED_CODE;
            CommonResponse commonResponse=new CommonResponse();
            commonResponse.setUrl(url);
            commonResponse.setResponseString(e.getMessage());
            message.obj=commonResponse;
            mHandler.sendMessage(message);
            return;
        }


        mOkHttpClient.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                Message message = Message.obtain();
                message.what=FAILED_CODE;
                CommonResponse commonResponse=new CommonResponse();
                commonResponse.setUrl(url);
                commonResponse.setResponseString(e.getMessage());
                message.obj=commonResponse;
                mHandler.sendMessage(message);
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                Message message = Message.obtain();
                ResponseBody body = response.body();

                InputStream inputStream=null;
                FileOutputStream fileOutputStream=null;

                try {
                    long totalLength = body.contentLength();
                   /* if(totalLength<=0){
                        message.what=FAILED_CODE;
                        CommonResponse commonResponse=new CommonResponse();
                        commonResponse.setUrl(url);
                        commonResponse.setResponseString("下载出错，无法下载该文件");
                        message.obj=commonResponse;
                        mHandler.sendMessage(message);
                        return;
                    }*/
                     inputStream = body.byteStream();
                     fileOutputStream=new FileOutputStream(file);
                    byte[] buffer = new byte[2048];
                    int len = 0;
                    //int num = 0;
                    //DecimalFormat df = new DecimalFormat("#.0");
                    while ((len = inputStream.read(buffer)) != -1) {
                        //num += len;
                        fileOutputStream.write(buffer, 0, len);
                        /*double d =  (double)num / (double) totalLength;
                        String format = df.format(d*100);
                        Message progressMsg = Message.obtain();
                        CommonProgress commonProgress=new CommonProgress();
                        commonProgress.setProgress(format);
                        commonProgress.setUrl(url);
                        progressMsg.what=PREGRESS_CODE;
                        progressMsg.obj=commonProgress;
                        mHandler.sendMessage(progressMsg);*/
                    }

                    Message progressMsg = Message.obtain();
                    CommonProgress commonProgress=new CommonProgress();
                    commonProgress.setProgress("100");
                    commonProgress.setUrl(url);
                    progressMsg.what=PREGRESS_CODE;
                    progressMsg.obj=commonProgress;
                    mHandler.sendMessage(progressMsg);
                } catch (Exception e){
                    message.what=FAILED_CODE;
                    CommonResponse commonResponse=new CommonResponse();
                    commonResponse.setUrl(url);
                    commonResponse.setResponseString("下载出错 "+e.getMessage());
                    message.obj=commonResponse;
                    mHandler.sendMessage(message);
                }finally {
                        if(inputStream!=null){
                            inputStream.close();
                        }
                    if(fileOutputStream!=null){
                        fileOutputStream.close();
                    }

                }
                mHandler.sendMessage(message);
            }
        });

    }
    public void downLoadFileFromServer(final String url, final File file, String method, BaseModule module) {
        this.mBaseModule=module;
        String baseUrl = "http://"+SharedPreferencesUtils.getIpAddress() + ":" + SharedPreferencesUtils.getPort();
        final String finalUrl = baseUrl+url;
        Request request;
        try{
            if(method.equals(Constants.GET)){
                request = new Request.Builder()
                        .url(finalUrl)
                        .get()
                        .addHeader("Accept-Encoding", "identity")
                        .addHeader(Constants.QAUTH,User.TOKEN)
                        .build();


            }else {
                FormBody.Builder builder = new FormBody.Builder();
                request = new Request.Builder()
                        .url(finalUrl)
                        .addHeader(Constants.QAUTH,User.TOKEN)
                        .addHeader("Accept-Encoding", "identity")
                        .post(builder.build())
                        .build();
            }
        }catch (Exception e){
            Message message = Message.obtain();
            message.what=FAILED_CODE;
            CommonResponse commonResponse=new CommonResponse();
            commonResponse.setUrl(url);
            commonResponse.setResponseString(e.getMessage());
            message.obj=commonResponse;
            mHandler.sendMessage(message);
            return;
        }

        mOkHttpClient.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                Message message = Message.obtain();
                message.what=FAILED_CODE;
                CommonResponse commonResponse=new CommonResponse();
                commonResponse.setUrl(url);
                commonResponse.setResponseString(e.getMessage());
                message.obj=commonResponse;
                mHandler.sendMessage(message);
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                Message message = Message.obtain();
                ResponseBody body = response.body();

                InputStream inputStream=null;
                FileOutputStream fileOutputStream=null;

                try {
                    long totalLength = body.contentLength();
                    Log.i("downloadProgress","totalLength===  "+totalLength);
                    inputStream = body.byteStream();
                    fileOutputStream=new FileOutputStream(file);
                    byte[] buffer = new byte[2048];
                    int len = 0;
                    int num = 0;
                    DecimalFormat df = new DecimalFormat("#.0");
                    while ((len = inputStream.read(buffer)) != -1) {
                        num += len;
                        fileOutputStream.write(buffer, 0, len);
                        double d =  (double)num / (double) totalLength;
                        String format = df.format(d*100);
                        Message progressMsg = Message.obtain();
                        CommonProgress commonProgress=new CommonProgress();
                        commonProgress.setProgress(format);
                        commonProgress.setUrl(url);
                        progressMsg.what=PREGRESS_CODE;
                        progressMsg.obj=commonProgress;
                        mHandler.sendMessage(progressMsg);
                    }
                } catch (Exception e){
                    message.what=FAILED_CODE;
                    CommonResponse commonResponse=new CommonResponse();
                    commonResponse.setUrl(url);
                    commonResponse.setResponseString("下载出错 "+e.getMessage());
                    message.obj=commonResponse;
                    mHandler.sendMessage(message);
                }finally {
                    if(inputStream!=null){
                        inputStream.close();
                    }
                    if(fileOutputStream!=null){
                        fileOutputStream.close();
                    }

                }
                mHandler.sendMessage(message);
            }
        });

    }
    public void uploadFileToServer(final String url, File file, Map<String, String> map, BaseModule module) {
        this.mBaseModule=module;
        String baseUrl = "http://"+SharedPreferencesUtils.getIpAddress() + ":" + SharedPreferencesUtils.getPort();
        final String finalUrl = baseUrl+url;
        Request request = null;
        try{
            MultipartBody.Builder builder = new MultipartBody.Builder();
            builder.setType(MultipartBody.FORM);

                if(file.exists()){
                    RequestBody requestBody=RequestBody.create(MediaType.parse("application/octet-stream"),file);
                    builder.addFormDataPart("file",file.getName(),requestBody);

                    for (Map.Entry<String, String> entry : map.entrySet()) {
                        builder.addFormDataPart(entry.getKey().trim(),entry.getValue().trim());

                    }
                }
            request = new Request.Builder()
                    .url(finalUrl)
                    .post(builder.build())
                    .addHeader(Constants.QAUTH,User.TOKEN)
                    .build();
        }catch (Exception e){
            Message message = Message.obtain();
            message.what=FAILED_CODE;
            CommonResponse commonResponse=new CommonResponse();
            commonResponse.setUrl(url);
            commonResponse.setResponseString(e.getMessage());
            message.obj=commonResponse;
            mHandler.sendMessage(message);
            return;
        }

        mOkHttpClient.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                Message message = Message.obtain();
                message.what=FAILED_CODE;
                CommonResponse commonResponse=new CommonResponse();
                commonResponse.setUrl(url);
                commonResponse.setResponseString(e.getMessage());
                message.obj=commonResponse;
                mHandler.sendMessage(message);
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                Message message = Message.obtain();

                String resp = response.body().string();
                if(TextUtils.isEmpty(resp)){
                    message.what=FAILED_CODE;
                    CommonResponse commonResponse=new CommonResponse();
                    commonResponse.setUrl(url);
                    commonResponse.setResponseString("服务器响应内容为空");
                    message.obj=commonResponse;
                    mHandler.sendMessage(message);
                    return;
                }
                try {
                    BaseEntity baseEntity = GsonUtils.gson(resp, BaseEntity.class);
                    //虽然响应成功，有可能数据不对
                    boolean status = baseEntity.isStatus();
                    int code = baseEntity.getCode();
                    if(status && code == 0){
                        message.what=SUCCESS_CODE;
                    }else {
                        message.what=FAILED_CODE;
                    }
                    CommonResponse commonResponse=new CommonResponse();
                    commonResponse.setUrl(url);
                    commonResponse.setResponseString(resp);
                    message.obj=commonResponse;
                } catch (JsonSyntaxException e){
                    try{
                        ErrorEntity errorEntity = GsonUtils.gson(resp, ErrorEntity.class);
                        message.what=FAILED_CODE;
                        CommonResponse commonResponse=new CommonResponse();
                        commonResponse.setUrl(url);
                        commonResponse.setResponseString("JsonSyntaxException "+errorEntity.toString());
                        message.obj=commonResponse;
                    }catch (JsonSyntaxException e1){
                        message.what=FAILED_CODE;
                        CommonResponse commonResponse=new CommonResponse();
                        commonResponse.setUrl(url);
                        commonResponse.setResponseString("JsonSyntaxException"+e1.getMessage());
                        message.obj=commonResponse;
                    }
                }
                mHandler.sendMessage(message);
            }
        });

    }
    public void uploadFileToServer(final String url, List<File> fileList,String type, BaseModule module) {
        this.mBaseModule=module;
        String baseUrl = "http://"+SharedPreferencesUtils.getIpAddress() + ":" + SharedPreferencesUtils.getPort();
        final String finalUrl = baseUrl+url;
        Request request = null;
        try{
            MultipartBody.Builder builder = new MultipartBody.Builder();
            builder.setType(MultipartBody.FORM);

            for (File f : fileList) {
                if(f.exists()){
                    RequestBody requestBody=RequestBody.create(MediaType.parse("application/octet-stream"),f);
                    builder.addFormDataPart("file",f.getName(),requestBody);
                    builder.addFormDataPart("type",type);
                }
            }
            request = new Request.Builder()
                    .url(finalUrl)
                    .post(builder.build())
                    .build();
        }catch (Exception e){
            Message message = Message.obtain();
            message.what=FAILED_CODE;
            CommonResponse commonResponse=new CommonResponse();
            commonResponse.setUrl(url);
            commonResponse.setResponseString(e.getMessage());
            message.obj=commonResponse;
            mHandler.sendMessage(message);
            return;
        }

        mOkHttpClient.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                Message message = Message.obtain();
                message.what=FAILED_CODE;
                CommonResponse commonResponse=new CommonResponse();
                commonResponse.setUrl(url);
                commonResponse.setResponseString(e.getMessage());
                message.obj=commonResponse;
                mHandler.sendMessage(message);
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                Message message = Message.obtain();
                String resp = response.body().string();
                if(TextUtils.isEmpty(resp)){
                    message.what=FAILED_CODE;
                    CommonResponse commonResponse=new CommonResponse();
                    commonResponse.setUrl(url);
                    commonResponse.setResponseString("服务器响应内容为空");
                    message.obj=commonResponse;
                    mHandler.sendMessage(message);
                    return;
                }
                try {
                    BaseEntity baseEntity = GsonUtils.gson(resp, BaseEntity.class);
                    //虽然响应成功，有可能数据不对
                    boolean status = baseEntity.isStatus();
                    int code = baseEntity.getCode();
                    if(status && code == 0){
                        message.what=SUCCESS_CODE;
                    }else {
                        message.what=FAILED_CODE;
                    }
                    CommonResponse commonResponse=new CommonResponse();
                    commonResponse.setUrl(url);
                    commonResponse.setResponseString(resp);
                    message.obj=commonResponse;
                } catch (JsonSyntaxException e){
                    try{
                        ErrorEntity errorEntity = GsonUtils.gson(resp, ErrorEntity.class);
                        message.what=FAILED_CODE;
                        CommonResponse commonResponse=new CommonResponse();
                        commonResponse.setUrl(url);
                        commonResponse.setResponseString("JsonSyntaxException "+errorEntity.toString());
                        message.obj=commonResponse;
                    }catch (JsonSyntaxException e1){
                        message.what=FAILED_CODE;
                        CommonResponse commonResponse=new CommonResponse();
                        commonResponse.setUrl(url);
                        commonResponse.setResponseString("JsonSyntaxException"+e1.getMessage());
                        message.obj=commonResponse;
                    }
                }
                mHandler.sendMessage(message);
            }
        });

    }

    public void uploadFileByURL(Map<String, String> map, final File file, final String url, String method, BaseModule module){
        this.mBaseModule=module;
        String baseUrl = "http://"+SharedPreferencesUtils.getIpAddress() + ":" + SharedPreferencesUtils.getPort();
        StringBuffer buffer=new StringBuffer();
        buffer.append("?");
        for (Map.Entry<String, String> entry : map.entrySet()) {
            buffer.append(entry.getKey().trim()+"="+entry.getValue().trim()+"&");
        }
        String s = buffer.toString();
        String parameter=s.substring(0,s.length()-1);
        final String finalUrl = baseUrl+url+parameter;

        new Thread(new Runnable() {
            @Override
            public void run() {
                FileOutputStream os=null;
                InputStream is=null;
                try {
                    URL mURL = new URL("http://192.168.0.129:8088/application/printApplication?cjxxbh=cjbh8343874990935552");
                    URLConnection conn = mURL.openConnection();
                   // conn.addRequestProperty();
                     is = conn.getInputStream();
                    int totalLength = conn.getContentLength();
                    byte[] buffer = new byte[1024];
                    int len;
                    int num=0;
                     os = new FileOutputStream(file);

                    DecimalFormat df = new DecimalFormat("#.0");
                    while ((len = is.read(buffer)) != -1) {
                        num += len;
                        os.write(buffer, 0, len);
                        double d =  (double)num / (double) totalLength;
                        String format = df.format(d*100);
                        Message progressMsg = Message.obtain();
                        CommonProgress commonProgress=new CommonProgress();
                        commonProgress.setProgress(format);
                        commonProgress.setUrl(url);
                        progressMsg.what=PREGRESS_CODE;
                        progressMsg.obj=commonProgress;
                        mHandler.sendMessage(progressMsg);
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }finally {
                    if(os!=null){
                        try {
                            os.close();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                    if(is!=null){
                        try {
                            is.close();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        }).start();

    }

}``
