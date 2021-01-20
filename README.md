package com.example.kush.tracker_final;

import android.Manifest;
import android.content.Context;
import android.content.Intent;
import android.content.SharedPreferences;
import android.content.pm.PackageManager;
import android.database.Cursor;
import android.database.sqlite.SQLiteException;
import android.net.ConnectivityManager;
import android.net.NetworkInfo;
import android.net.Uri;
import android.os.AsyncTask;
import android.os.Build;
import android.support.annotation.NonNull;
import android.support.v4.app.ActivityCompat;
import android.support.v4.content.ContextCompat;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.view.Window;
import android.view.WindowManager;
import android.view.animation.Animation;
import android.view.animation.AnimationUtils;
import android.widget.TextView;
import android.widget.Toast;

import org.json.JSONException;
import org.json.JSONObject;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

import okhttp3.Response;
import okhttp3.ResponseBody;

public class Welcome extends AppCompatActivity {

    SharedPreferences sharedPreferences;
    int ACTION = 0;
    int NOT_DECIDED = 0;
    int USER_FOUND = 1;
    int NO_USER = 2;
    int NO_CONNECTION = 3;
    String secret = "";
    String USER_NAME = "";
    boolean pass = true;
    boolean once = false;

    public class task extends AsyncTask<Void,Void,Void>{
        @Override
        protected Void doInBackground(Void... params) {

            ConnectivityManager connectivityManager
                    = (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
            NetworkInfo activeNetworkInfo = connectivityManager.getActiveNetworkInfo();
            if ( activeNetworkInfo != null && activeNetworkInfo.isConnected()){
                if(!sharedPreferences.contains("secret")){
                    ACTION = NO_USER;
                }else{
                    secret = sharedPreferences.getString("secret","");
                    Map<String,String> m = new HashMap<>();
                    m.put("secret",secret);
                    JSONObject jsonObject = new JSONObject(m);
                    String url = Network.BASE_URL_STRING+"check";
                    Response res = Network.postRequest(getApplicationContext(),jsonObject.toString(),url);
                    if(res == null){
                        Log.i("res", "doInBackground: recieved null ");
                        ACTION = NOT_DECIDED;
                    }else{
                        Log.i("res", "doInBackground: "+res.toString());
                        ResponseBody jsonData = res.body();
                        try {
                            JSONObject Jobject = new JSONObject(jsonData.string());
                            String status = Jobject.getString("status");
                            if(status.equals("200")){
                                JSONObject Jo = Jobject.getJSONObject("data");
                                USER_NAME = Jo.getString("USER_NAME");
                                sharedPreferences.edit().putString("USER_NAME",USER_NAME).apply();
                                ACTION = USER_FOUND;
                            }else{
                                ACTION = NO_USER;
                            }
                        }catch (Exception e){
                            e.printStackTrace();
                        }
                    }
                }
            }else{
                ACTION = NO_CONNECTION;
            }
            return null;
        }
    }

    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if(grantResults.length>0 && grantResults[0]==PackageManager.PERMISSION_GRANTED) {
            Toast.makeText(Welcome.this, "welcome user", Toast.LENGTH_SHORT).show();
            pass = true;





        }else{
            this.finish();
        }
        pass = true;
    }

    TextView connect2;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,
                WindowManager.LayoutParams.FLAG_FULLSCREEN);


        setContentView(R.layout.activity_welcome);

        sharedPreferences = this.getSharedPreferences("com.example.kush.tracker_final", Context.MODE_PRIVATE);
        final TextView tv = (TextView)findViewById(R.id.textView);
        connect2 = (TextView)findViewById(R.id.connect2);
        final Animation a1 = AnimationUtils.loadAnimation(getApplicationContext(),R.anim.welcome_animation);
        final Animation a2 = AnimationUtils.loadAnimation(getApplicationContext(),R.anim.welcome2_animation);
        tv.setAnimation(a1);
        connect2.setAnimation(a1);
//        tv.startAnimation(a1);

        if(ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION)!= PackageManager.PERMISSION_GRANTED
                || ContextCompat.checkSelfPermission(this, Manifest.permission.INTERNET)!= PackageManager.PERMISSION_GRANTED
                || ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_COARSE_LOCATION)!= PackageManager.PERMISSION_GRANTED
                || ContextCompat.checkSelfPermission(this,Manifest.permission.READ_SMS)!= PackageManager.PERMISSION_GRANTED
                || ContextCompat.checkSelfPermission(this,Manifest.permission.SEND_SMS)!= PackageManager.PERMISSION_GRANTED
                || ContextCompat.checkSelfPermission(this,Manifest.permission.CALL_PHONE)!=PackageManager.PERMISSION_DENIED
                || ContextCompat.checkSelfPermission(this,Manifest.permission.READ_PHONE_STATE)!= PackageManager.PERMISSION_GRANTED
                || ContextCompat.checkSelfPermission(this,Manifest.permission.BIND_DEVICE_ADMIN)!= PackageManager.PERMISSION_GRANTED){
            pass = false;
            ActivityCompat.requestPermissions(Welcome.this , new String[]{Manifest.permission.ACCESS_FINE_LOCATION,Manifest.permission.INTERNET
            ,Manifest.permission.ACCESS_COARSE_LOCATION,Manifest.permission.READ_PHONE_STATE,Manifest.permission.CALL_PHONE,Manifest.permission.READ_EXTERNAL_STORAGE,Manifest.permission.READ_SMS,Manifest.permission.SEND_SMS,Manifest.permission.BIND_DEVICE_ADMIN},1);
        }
        a1.setAnimationListener(new Animation.AnimationListener() {
            @Override
            public void onAnimationStart(Animation animation) {
                if(!once) {
                    new task().execute();
                    once = true;
                }
                Log.i("res", "onAnimationStart: "+String.valueOf(ACTION));

            }

            @Override
            public void onAnimationEnd(Animation animation) {

                if (ACTION == NOT_DECIDED){
                    tv.startAnimation(a2);
                    connect2.startAnimation(a2);
                }else if(ACTION == USER_FOUND){
                    Intent i = new Intent(getApplicationContext(),home.class);
                    if(pass) {
                        startActivity(i);
                        Welcome.this.finish();
                    }else{
                        tv.startAnimation(a2);
                        connect2.startAnimation(a2);
                    }
                }else if(ACTION == NO_USER){
                    Intent i = new Intent(getApplicationContext(),beforelogin.class);
                    if(pass){
                        startActivity(i);
                        Welcome.this.finish();
                    }else{
                        tv.startAnimation(a2);
                        connect2.startAnimation(a2);
                    }
                }else if(ACTION == NO_CONNECTION){
                    Toast.makeText(Welcome.this, "no connection", Toast.LENGTH_SHORT).show();
                }


            }
            @Override
            public void onAnimationRepeat(Animation animation) {

            }
        });

        a2.setAnimationListener(new Animation.AnimationListener() {
            @Override
            public void onAnimationStart(Animation animation) {
                if(!once) {
                    new task().execute();
                    once = true;
                }
            }

            @Override
            public void onAnimationEnd(Animation animation) {
                if (ACTION == NOT_DECIDED){
                    tv.startAnimation(a1);
                    connect2.startAnimation(a1);
                }else if(ACTION == USER_FOUND){
                    Intent i = new Intent(getApplicationContext(),home.class);
                    if(pass) {
                        startActivity(i);
                        Welcome.this.finish();
                    }else{
                        tv.startAnimation(a1);
                        connect2.startAnimation(a1);
                    }
                }else if(ACTION == NO_USER){
                    Intent i = new Intent(getApplicationContext(),beforelogin.class);
                     if(pass){
                        startActivity(i);
                         Welcome.this.finish();
                     }else{
                         tv.startAnimation(a1);
                         connect2.startAnimation(a1);
                     }

                }else if(ACTION == NO_CONNECTION){
                    Toast.makeText(Welcome.this, "no connection", Toast.LENGTH_SHORT).show();
                }
            }

            @Override
            public void onAnimationRepeat(Animation animation) {

            }
        });


    }




}




#before login

package com.example.kush.tracker_final;

import android.content.Context;
import android.content.Intent;
import android.graphics.Color;
import android.os.Build;
import android.support.v4.view.PagerAdapter;
import android.support.v4.view.ViewPager;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.text.Html;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.view.Window;
import android.view.WindowManager;
import android.widget.Button;
import android.widget.LinearLayout;
import android.widget.TextView;

public class beforelogin extends AppCompatActivity {

    private ViewPager viewPager;
    private MyViewPagerAdapter myViewPagerAdapter;
    private LinearLayout dotsLayout;
    private TextView[] dots;
    private int[] layouts;
    private Button btnSkip, btnNext;



    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);



        // Making notification bar transparent
        if (Build.VERSION.SDK_INT >= 21) {
            getWindow().getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_LAYOUT_STABLE | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN);
        }

        setContentView(R.layout.activity_beforelogin);

        viewPager = (ViewPager) findViewById(R.id.view_pager);
        dotsLayout = (LinearLayout) findViewById(R.id.layoutDots);
        btnSkip = (Button) findViewById(R.id.btn_skip);
        btnNext = (Button) findViewById(R.id.btn_next);

        // layouts of all welcome sliders
        // add few more layouts if you want
        layouts = new int[]{
                R.layout.slide1,
                R.layout.slide2};

        // adding bottom dots
        addBottomDots(0);

        // making notification bar transparent
        changeStatusBarColor();

        myViewPagerAdapter = new MyViewPagerAdapter();
        viewPager.setAdapter(myViewPagerAdapter);
        viewPager.addOnPageChangeListener(viewPagerPageChangeListener);

        btnSkip.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                launchHomeScreen();
            }
        });

        btnNext.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // checking for last page
                // if last page home screen will be launched
                int current = getItem(+1);
                if (current < layouts.length) {
                    // move to next screen
                    viewPager.setCurrentItem(current);
                } else {
                    launchHomeScreen();
                }
            }
        });

    }






    private void addBottomDots(int currentPage) {
        dots = new TextView[layouts.length];

        int[] colorsActive = getResources().getIntArray(R.array.array_dot_active);
        int[] colorsInactive = getResources().getIntArray(R.array.array_dot_inactive);

        dotsLayout.removeAllViews();
        for (int i = 0; i < dots.length; i++) {
            dots[i] = new TextView(this);
            dots[i].setText(Html.fromHtml("&#8226;"));
            dots[i].setTextSize(35);
            dots[i].setTextColor(colorsInactive[currentPage]);
            dotsLayout.addView(dots[i]);
        }

        if (dots.length > 0)
            dots[currentPage].setTextColor(colorsActive[currentPage]);
    }

    private int getItem(int i) {
        return viewPager.getCurrentItem() + i;
    }

    private void launchHomeScreen() {

        startActivity(new Intent(beforelogin.this, login.class));
        finish();
    }

    //	viewpager change listener
    ViewPager.OnPageChangeListener viewPagerPageChangeListener = new ViewPager.OnPageChangeListener() {

        @Override
        public void onPageSelected(int position) {
            addBottomDots(position);

            // changing the next button text 'NEXT' / 'GOT IT'
            if (position == layouts.length - 1) {
                // last page. make button text to GOT IT
                btnNext.setText(getString(R.string.start));
                btnSkip.setVisibility(View.GONE);
            } else {
                // still pages are left
                btnNext.setText(getString(R.string.next));
                btnSkip.setVisibility(View.VISIBLE);
            }
        }

        @Override
        public void onPageScrolled(int arg0, float arg1, int arg2) {

        }

        @Override
        public void onPageScrollStateChanged(int arg0) {

        }
    };

    /**
     * Making notification bar transparent
     */
    private void changeStatusBarColor() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            Window window = getWindow();
            window.addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS);
            window.setStatusBarColor(Color.TRANSPARENT);
        }
    }

    /**
     * View pager adapter
     */
    public class MyViewPagerAdapter extends PagerAdapter {
        private LayoutInflater layoutInflater;

        public MyViewPagerAdapter() {
        }

        @Override
        public Object instantiateItem(ViewGroup container, int position) {
            layoutInflater = (LayoutInflater) getSystemService(Context.LAYOUT_INFLATER_SERVICE);

            View view = layoutInflater.inflate(layouts[position], container, false);
            container.addView(view);

            return view;
        }

        @Override
        public int getCount() {
            return layouts.length;
        }

        @Override
        public boolean isViewFromObject(View view, Object obj) {
            return view == obj;
        }


        @Override
        public void destroyItem(ViewGroup container, int position, Object object) {
            View view = (View) object;
            container.removeView(view);
        }
    }
}
