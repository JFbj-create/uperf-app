package com.uperf.app;

import android.annotation.SuppressLint;
import android.os.Bundle;
import android.os.Handler;
import android.os.Looper;
import android.view.View;
import android.webkit.WebChromeClient;
import android.webkit.WebResourceError;
import android.webkit.WebResourceRequest;
import android.webkit.WebSettings;
import android.webkit.WebView;
import android.webkit.WebViewClient;
import android.widget.Button;
import android.widget.LinearLayout;
import android.widget.ProgressBar;
import android.widget.TextView;

import androidx.appcompat.app.AppCompatActivity;

import java.io.BufferedReader;
import java.io.File;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;

/**
 * Uperf 涓荤晫闈?- WebView 鍖呰鍣? *
 * 鍚姩娴佺▼:
 *   1. 妫€娴?Web API (http://127.0.0.1:8765/) 鏄惁鍦ㄧ嚎 (2绉掕秴鏃?
 *   2. 鍦ㄧ嚎 -> 鍔犺浇 Web UI
 *   3. 绂荤嚎 -> 妫€娴嬫湰鍦版枃浠?/sdcard/uperf_web/index.html
 *      - 瀛樺湪 -> 鍔犺浇鏈湴鏂囦欢 (API 涓嶅彲鐢ㄤ絾 UI 鍙湅)
 *      - 涓嶅瓨鍦?-> 鏄剧ず "妯″潡鏈摼鎺? 閿欒椤? *
 * 鍚屾椂灏濊瘯鐢?root 璇诲彇 /data/adb/modules/uperf/module.prop
 * 鐢ㄤ簬鍖哄垎 "妯″潡鏈畨瑁? 鍜?"妯″潡宸插畨瑁呬絾鏈嶅姟鏈惎鍔?
 */
public class MainActivity extends AppCompatActivity {

    private static final String WEB_URL = "http://127.0.0.1:8765/";
    private static final String LOCAL_FALLBACK = "/sdcard/uperf_web/index.html";
    private static final String MODULE_PROP = "/data/adb/modules/uperf/module.prop";
    private static final String MODULE_PATH = "/data/adb/modules/uperf";

    private WebView webView;
    private ProgressBar progressBar;
    private LinearLayout errorView;
    private TextView errorText;
    private Button refreshBtn;

    private final Handler handler = new Handler(Looper.getMainLooper());

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        webView = findViewById(R.id.webview);
        progressBar = findViewById(R.id.progress_bar);
        errorView = findViewById(R.id.error_view);
        errorText = findViewById(R.id.error_text);
        refreshBtn = findViewById(R.id.refresh_btn);
        Button retryBtn = findViewById(R.id.retry_btn);
        Button openLocalBtn = findViewById(R.id.open_local_btn);

        setupWebView();

        refreshBtn.setOnClickListener(v -> checkAndLoad());
        retryBtn.setOnClickListener(v -> checkAndLoad());
        openLocalBtn.setOnClickListener(v -> loadLocalFallback());

        checkAndLoad();
    }

    @SuppressLint("SetJavaScriptEnabled")
    private void setupWebView() {
        WebSettings settings = webView.getSettings();
        // Web API 渚濊禆 JavaScript
        settings.setJavaScriptEnabled(true);
        settings.setDomStorageEnabled(true);
        settings.setAllowFileAccess(true);
        settings.setAllowContentAccess(true);
        settings.setDatabaseEnabled(true);
        settings.setCacheMode(WebSettings.LOAD_DEFAULT);
        settings.setSupportZoom(true);
        settings.setBuiltInZoomControls(true);
        settings.setDisplayZoomControls(false);
        settings.setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);

        webView.setWebViewClient(new WebViewClient() {
            @Override
            public void onReceivedError(WebView view, WebResourceRequest request, WebResourceError error) {
                super.onReceivedError(view, request, error);
                if (request.isForMainFrame()) {
                    showError("Web 椤甸潰鍔犺浇澶辫触\n\n" + error.getDescription());
                }
            }
        });

        webView.setWebChromeClient(new WebChromeClient() {
            @Override
            public void onProgressChanged(WebView view, int newProgress) {
                super.onProgressChanged(view, newProgress);
                progressBar.setProgress(newProgress);
                progressBar.setVisibility(newProgress < 100 ? View.VISIBLE : View.GONE);
            }
        });
    }

    /** 寮傛妫€娴嬫ā鍧楃姸鎬佸苟鍔犺浇瀵瑰簲鐣岄潰 */
    private void checkAndLoad() {
        showLoading();
        new Thread(() -> {
            boolean webOk = checkWebRunning();
            boolean moduleInstalled = checkModuleInstalled();
            boolean localExists = new File(LOCAL_FALLBACK).exists();

            handler.post(() -> {
                if (webOk) {
                    loadWebUrl();
                } else if (moduleInstalled && localExists) {
                    loadLocalFallback();
                } else if (moduleInstalled) {
                    showError("妯″潡宸插畨瑁呬絾 Web 鏈嶅姟鏈惎鍔╘n\n璇烽噸鍚澶囨垨鍦ㄧ粓绔墽琛?\nsh /data/adb/modules/uperf/run_uperf.sh");
                } else {
                    showError("妯″潡鏈摼鎺n\n璇峰厛瀹夎 uperf Magisk 妯″潡\n瀹夎瀹屾垚鍚庨噸鍚澶囧嵆鍙娇鐢?);
                }
            });
        }).start();
    }

    /** 妫€娴?Web API 鏄惁鍦ㄧ嚎 (HTTP 200) */
    private boolean checkWebRunning() {
        try {
            URL url = new URL(WEB_URL);
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            conn.setConnectTimeout(2000);
            conn.setReadTimeout(2000);
            conn.setRequestMethod("GET");
            int code = conn.getResponseCode();
            conn.disconnect();
            return code == 200;
        } catch (Exception e) {
            return false;
        }
    }

    /** 妫€娴?uperf 妯″潡鏄惁宸插畨瑁?(root + 鏂囦欢妫€娴? */
    private boolean checkModuleInstalled() {
        // 鏂瑰紡 1: root 璇诲彇 module.prop
        try {
            Process p = Runtime.getRuntime().exec(new String[]{"su", "-c", "cat " + MODULE_PROP});
            BufferedReader reader = new BufferedReader(new InputStreamReader(p.getInputStream()));
            String line = reader.readLine();
            p.waitFor();
            if (line != null && (line.contains("id=") || line.contains("name="))) {
                return true;
            }
        } catch (Exception e) {
            // su 涓嶅彲鐢?缁х画灏濊瘯鍏朵粬鏂瑰紡
        }
        // 鏂瑰紡 2: 妫€鏌ユ湰鍦?Web 鏂囦欢 (妯″潡瀹夎鏃朵細澶嶅埗鍒?/sdcard/uperf_web/)
        if (new File(LOCAL_FALLBACK).exists()) {
            return true;
        }
        // 鏂瑰紡 3: root 妫€鏌ユā鍧楃洰褰?        try {
            Process p = Runtime.getRuntime().exec(new String[]{"su", "-c", "test -d " + MODULE_PATH + " && echo yes"});
            BufferedReader reader = new BufferedReader(new InputStreamReader(p.getInputStream()));
            String line = reader.readLine();
            p.waitFor();
            return "yes".equals(line);
        } catch (Exception e) {
            return false;
        }
    }

    private void loadWebUrl() {
        webView.setVisibility(View.VISIBLE);
        errorView.setVisibility(View.GONE);
        progressBar.setIndeterminate(false);
        webView.loadUrl(WEB_URL);
    }

    private void loadLocalFallback() {
        webView.setVisibility(View.VISIBLE);
        errorView.setVisibility(View.GONE);
        progressBar.setIndeterminate(false);
        File f = new File(LOCAL_FALLBACK);
        if (f.exists()) {
            webView.loadUrl("file://" + LOCAL_FALLBACK);
        } else {
            showError("鏈湴 Web 鏂囦欢涓嶅瓨鍦╘n\n" + LOCAL_FALLBACK);
        }
    }

    private void showLoading() {
        webView.setVisibility(View.GONE);
        errorView.setVisibility(View.GONE);
        progressBar.setVisibility(View.VISIBLE);
        progressBar.setIndeterminate(true);
    }

    private void showError(String msg) {
        webView.setVisibility(View.GONE);
        progressBar.setVisibility(View.GONE);
        errorView.setVisibility(View.VISIBLE);
        errorText.setText(msg);
    }

    @Override
    public void onBackPressed() {
        if (webView.getVisibility() == View.VISIBLE && webView.canGoBack()) {
            webView.goBack();
        } else {
            super.onBackPressed();
        }
    }
}