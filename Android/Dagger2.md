## Dagger2  

+ Inject
+ Module
+ Component
+ DaggerXXXComponent.inject


```java
引入Dagger库，如果使用Kotlin，还需要将annotationProcessor改成kapt

 implementation 'com.google.dagger:dagger:2.20'
//    annotationProcessor 'com.google.dagger:dagger-compiler:2.20'
    kapt 'com.google.dagger:dagger-compiler:2.20'

    implementation 'com.google.dagger:dagger-android:2.20'
    implementation 'com.google.dagger:dagger-android-support:2.20'
//    annotationProcessor 'com.google.dagger:dagger-android-processor:2.20'
    kapt 'com.google.dagger:dagger-android-processor:2.20'

```
  


### Inject  


可以用来标识**变量**和**构造函数**，当然也可以标记方法，不过很少用到

```java
// 标识变量，告诉Dagger，该类需要ViewModelFactory对象
@Inject
lateinit mFactory: ViewModelFactory

// 标识构造函数，告诉Dagger，可以提供LoginViewModel对象
@Inject
public LoginViewModel(){
}
```    

**注意**：

1. `@Inject`一般标记成员属性和构造函数，标记的成员属性不能是*private*


### Component  

用于标注接口或抽象类，称之为注射器，负责将LoginViewModel提供的依赖，注入了所需要的地方；

Component就是连接**依赖的对象实例**和**需要注入的实例属性**之间的桥梁

```java
@Component(modules = LoginModule::class)
interface ActivityComponent{
	fun inject(activity: LoginActivity)  // 这里只能是LoginActivity，不能是LoginActivity的父类或子类
}
```


### Module  

也是用来提供依赖的，既然有Inject，为什么还需要Module?  

**原因**：

使用`@Inject`标记构造函数来提供依赖的对象实例，不是万能的，以下场景无法使用：

+ 接口没有构造函数
+ 第三方路的类不能被标记
+ 构造函数中的参数也必须配置


**注意**：

问题1： 如果既在构造函数上标记`@Inject`提供依赖，同时又通过`@Module`提供依赖，那么Dagger2会如何选择呢？

答：  `@Module`优先级高于`@Inject`

1. 首先查找`@Module`标注的类中是否存在提供依赖的方法
2. 若存在提供依赖的方法，查看该方法是否含有参数
	+ 若存在参数，则从步骤1开始一次初始化每一个参数
	+ 若不存在，则直接初始化该类的实例，完成一次依赖注入
3. 若不存在提供依赖的方法，则查找`@Inject`标记的构造函数，看构造函数是否存在参数
	+ 若存在参数，则从步骤1开始初始化每一个参数
	+ 若不存在，则直接初始化该实例，完成依赖注入



### Provides  

在Module中使用，通常标记方法，该方法在需要提供依赖的时候被调用

### Scope  

用于自定义注解，可以通过`@Scope`自定义注解来限定作用域，实现局部的单例，例如实现Component内的单例，其实就是让生成的依赖实例的生命周期与Component绑定；

如果没有使用Scope注解，Component每次调用Module中的provide方法或Inject构造函数时都会创建一个新的实例，而使用Scope后可以复用之前的依赖实例。


`@Scope`是元注解，用来标记自定义注解的，ActivityScope就是一个Scope注解，Scope注解只能标记类、provide方法以及Component。

生效条件：**同时**标记在Component和提供依赖实例的Module上，且Scope必须得相同作用域  

通过DoubleCheck保证只会生成一次实例

```java
@MustBeDocumented
@Retention(AnnotationRetention.RUNTIME)
@Scope
annotation class ActivityScope
```



### Lazy   

延迟注入，依赖在使用的时候再完成初始化，加快加载速度，只有在调用`Lazy`的`get()`的方法时才会初始化依赖实例，注入依赖

### Qulifier、Named  

如果在Module中提供两个生成类A实例的provide方法，Dagger2是不知道使用哪一个的。

可以自定义`@Named`注解，  

```java 

@Qualifier
@Documented
@Retention(RUNTIME)
public @interface Named {
	String value() default "";
}


@Provides
@Named("a1")
A provideA(){
	return new A();
}
```


***   

### 使用方式   



#### 方式一（手动inject）  

```java

class RegisterActivity : BaseActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
    
        DaggerRegisterComponent.builder().build().inject(this)
        
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_register)
    }
}
```  

**缺点**： 

1. 几乎每个类中都需要手动通过DaggerXXXComponent的inject的方式去注入，模板代码不简洁，导致重构困难
2. 打破依赖注入原则，类不应该知道它本身是如何被注入的

**注**：详见develop分支，RegisterActivity的实现


#### 方式二（ContributesAndroidInjector）  


App继承DaggerApplication  

```java
class App : DaggerApplication() {
    override fun applicationInjector(): AndroidInjector<out DaggerApplication> {
        return DaggerAppComponent.create()
    }
}
```


创建用于生成Activity注入器的Module，可以管理各个Activity对应的module

```java  
@Module
abstract class ActivityBindingModule {

    @ActivityScope
    @ContributesAndroidInjector(modules = [LoginModule::class])
    abstract fun contributeLoginActivity(): LoginActivity
}
```    

创建AppComponent，继承AndroidInjector，指定Application，将ActivityBindingModule和Dagger2提供的AndroidSupportInjectionModule作为module

```java  

@Component(modules = [ActivityBindingModule::class, AndroidSupportInjectionModule::class])
interface AppComponent : AndroidInjector<App> {

}
```  

LoginActivity继承DaggerAppCompatActivity，在onCreate方法中调用AndroidInjection.inject(this)，当然可以封装到父类中  

```java  
class LoginActivity : DaggerAppCompatActivity() {

    private lateinit var mLoginViewModel: LoginViewModel

    @Inject
    lateinit var mFactory: LoginViewModelFactory

    override fun onCreate(savedInstanceState: Bundle?) {
        AndroidInjection.inject(this)  
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_login)
    }    
}
```   

#### TODO  

1. 目前每一个ViewModel都需要创建一个对应的ViewModelFactory，后续优化（通过IntoMap，只创建一个通用的Factory），done，参考**WanAndroid项目**


#### 参考文献   
[dagger & android](https://dagger.dev/android.html)  

[Dagger2在Android上的使用](https://yuweiguocn.github.io/dagger2-1/)  

[Dagger2与AndroidInjector](https://blog.csdn.net/qq_17766199/article/details/73030696)   





