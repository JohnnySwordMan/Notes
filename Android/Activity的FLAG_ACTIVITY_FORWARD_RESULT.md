# Activity的FLAG\_ACTIVITY\_FORWARD_RESULT

### 场景
最近项目中新增一个需求，需要在多个Activity之间传值。例如，ProjectActivity页面到BuyActivtity页面，最后到BuyResultActivity页面，BuyResultAcitity页面finish后需要将值传递给ProjectActivity。

### 实现
通过利用Intent的*FLAG\_ACTIVITY\_FORWARD_RESULT*标志位即可，即使在ProjectActivity和BuyResultActivity之间有许多页面过渡也能实现。

##### ProjectActivity跳转BuyActivity   

```java
public class ProjectActivity extends AppCompatActivity {

    private TextView mToDetailActivity;
    private static final int REQUEST_CODE_INVEST = 1000;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_project);

        mToDetailActivity = findViewById(R.id.tv_to_buy_activity);

        mToDetailActivity.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent(ProjectActivity.this,BuyActivity.class);
                startActivityForResult(intent,REQUEST_CODE_INVEST);
            }
        });
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if(requestCode == REQUEST_CODE_INVEST){
            if(data != null){
                Log.d("JayGe","ProjectActivity, amount = "+data.getIntExtra("amount",100));
            } else {
                Log.d("JayGe","ProjectActivity, data为null");
            }
        }
    }
}  

```

##### BuyActivity跳转BuyResultActivity，BuyActivity可看作过渡页面  

```java
public class BuyActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_buy);

        findViewById(R.id.tv_to_buy_result_activity).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent(BuyActivity.this, BuyResultActivity.class);
                intent.setFlags(Intent.FLAG_ACTIVITY_FORWARD_RESULT);
                startActivity(intent);
                finish();
            }
        });
    }
}
```

##### BuyResultActivity操作结束后，将值传递回ProjectActivity  

```java
public class BuyResultActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_buy_result);

        findViewById(R.id.tv_return_data_activity).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = getIntent();
                intent.putExtra("amount",99);
                setResult(RESULT_OK,intent);
                finish();
            }
        });
    }
}
```

### 总结
通过Intent的*FLAG\_ACTIVITY\_FORWARD_RESULT*可以将最后一个页面的值回传给第一个页面。
另：页面之间传值的方式有很多种，本文的传值方式是项目中用到的一种特殊的需求，在此记录下。

