### Fragment对象的创建   


我们可以通过new Fragment的方式创建Fragment对象，代码如下：

`DepositFragment fragment = new DepositFragment();`

很多情况下，创建Fragment对象的时候，会传入参数，因此可以使用带参的构造函数，又或者在Fragment类中定义一个方法设置数据，代码如下：`DepositFragment fragment = new DepositFragment(100);`或`DepositFragment fragment = new DepositFragment();
fragment.setData(100);`

一切看似正常，但是在特殊情况下还是会产生问题。
在手机内存不足的时候，系统可能会回收Activity并重新创建，因此依附于Activity的Fragment也会重新创建，但是此时会默认调用无参的构造函数，可能会导致空指针异常或者无数据。

因此创建Fragment实例对象的最好的写法如下：  

```java
public static DepositCycleListFragment newInstance(String response) {
    Bundle args = new Bundle();
    args.putString("key_response",response);
    DepositCycleListFragment fragment = new DepositCycleListFragment();
    fragment.setArguments(args);
    return fragment;
}

@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    Bundle arguments = getArguments();
    if(arguments != null){
        String response = arguments.getString("key_response");
    }
}
```  


### 显示/隐藏Fragment
1. `add()`和`replace()`,`show()`和`hide()`  
添加Fragment，可以使用add()和replace()，两者的区别是add()方法会将新的Fragment覆盖在原先的Fragment上，而replace()方法则是替换之前的Fragment，会执行onDestoryView方法等。
show()和hide()相当于让Fragment可见或是不可见，并不会触发生命周期的执行。

2. 使用场景
如果当前的Fragment重复出现的几率较大，建议使用show()、hide()，这样可以避免replace()导致的Fragment的重复创建，提高App的性能。
以点击底部Tab切换Fragment为例，标准写法：  

```java
// 点击首页和我的Tab切换Fragment
 private void changeTab(int index) {
        // 通过tag来标记是否已选中某个tab
        mCurrTabIndex = index;
        FragmentManager fm = getSupportFragmentManager();
        FragmentTransaction transaction = fm.beginTransaction();
        hideFragments(transaction, index);
        addOrShowFragment(transaction, index);
        transaction.commitAllowingStateLoss();
        // 设置Tab的选中效果
        setTabSelected(index);
    }
    
    // 隐藏Fragment
     private void hideFragments(FragmentTransaction ft, int exclusiveIndex) {
        if (exclusiveIndex != 0 && homeFragment != null){
            ft.hide(homeFragment);
        }
        if (exclusiveIndex != 1 && meFragment != null){
            ft.hide(meFragment);
        }
    }
    
    // 显示Fragment
    private void addOrShowFragment(FragmentTransaction ft, int index) {
        if (index == 0) {
            if (homeFragment == null) {
                homeFragment = HomeFragment.newInstance();
                ft.add(R.id.container_layout, homeFragment, TAB_HOME);
            } else {
                ft.show(homeFragment);
            }
        } else if(index == 1){
            if (meFragment == null) {
                meFragment = MeFragment.newInstance();
                ft.add(R.id.container_layout, meFragment, TAB_ME);
            } else {
                ft.show(meFragment);
            }
        }
    }
```

### onHiddenChanged() 

一般情况下，我们可能会在onCreate()或onResume()方法中请求Fragment的数据，但是show()、hide()方法并不会触发Fragment的生命周期，因此对于有的需要在切换Fragment时重新请求网络数据的需求，我们单单仅仅只在onResume()fang方法中请求网络数据是不够的，我们该如何做呢？   


使用show()和hide()方法切换Fragment时，会回调onHiddenChanged()方法，注意：新Fragment在创建的时候不会回调onHiddenChanged()方法。    


考虑三种情况：    

+ 第一种，HomeFragment与MeFragment已经显示过的情况下，切换Fragment，需要在onHiddenChanged()方法中请求网络数据；  
+ 第二种，从其他Activity返回至当前Fragment，当前Fragment不会调用onHiddenChanged()方法，需要在onResume()方法中请求网络数据；  
+ 第三种，从其他Activity返回，同时切换到处于隐藏状态的Fragment，则onResume()和onHiddenChanged()方法都会执行。  
代码示例如下：  

    ```java
    @Override
    public void onResume() {
        super.onResume();
        if (!isHidden()) {
            // 如果可见，发起网络请求
            loadData();
        }
    }
    
    @Override
    public void onHiddenChanged(boolean hidden) {
        super.onHiddenChanged(hidden);
        if (!hidden && isResumed()) {
            // 如果可见
            // 判断isResumed()为了避免与onResume()方法重复发起网络请求
            loadData();
        }
    }
    
    @Override
    public void onCreate(Bundle savedInstanceState){
        // 只执行一次，由于onResume()方法中可见才会请求网络数据，
        // 可能会出现页面已经显示，但是数据还没有的情况，为了优化体验，最好在onCreate()方法中也做仅做一次网络请求
        if(firstInit){
            loadData();
        }
    }
    ```  
    
    
### 文献参考
[Android Fragment 的使用，一些你不可不知的注意事项](http://yifeng.studio/2016/12/15/android-fragment-attentions/)