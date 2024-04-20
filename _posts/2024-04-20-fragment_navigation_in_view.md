---
title: Fragment Navigation in View

---
# 1.实现跳转

## 1.1 在需要实现跳转的主界面如ActivityMain.xml内，写fragment  

```xml
<fragment  
    android:id="@+id/nav_host_fragment_activity_main"  
  
    android:name="androidx.navigation.fragment.NavHostFragment"  
  
    android:layout_width="match_parent"  
    android:layout_height="0dp"  
    app:defaultNavHost="true"  
    app:layout_constraintBottom_toTopOf="@id/nav_view"  
    app:layout_constraintHorizontal_bias="0.0"  
    app:layout_constraintLeft_toLeftOf="parent"  
    app:layout_constraintRight_toRightOf="parent"  
    app:layout_constraintTop_toBottomOf="@id/imagemain"  
    app:layout_constraintVertical_bias="0.0"  
    app:navGraph="@navigation/mobile_navigation" />
```
写出fragment的要点
1. 要记住id，findNavigation的时候要id
2. 这个name里面的NavHostFragment是核心，对初始的跳转页要写这个
3. 写navGraph，图给到跳转的层级，里面定义了各个跳转动作

## 1.2 主界面里写底部导航栏

ActivityMain.xml内写底部导航栏的内容
bottom_nav_menu是一个xml文件，定义了menu内的item，定义名字、icon  
```xml
<com.google.android.material.bottomnavigation.BottomNavigationView  
    android:id="@+id/nav_view"  
    android:layout_width="0dp"  
    android:layout_height="wrap_content"  
    android:background="?android:attr/windowBackground"  
    app:layout_constraintBottom_toBottomOf="parent"  
    app:layout_constraintLeft_toLeftOf="parent"  
    app:layout_constraintRight_toRightOf="parent"  
    app:menu="@menu/bottom_nav_menu" />
```

## 1.3 绑定导航栏与navController

通过AppBarConfiguration写setOf定义顶层图（在这些图内不会自动配备appbar内的返回），如果传入graph进去会自动判断，但是有并列等级啥的问题
绑定navController与顶层视图，R.id.nav_host_fragment_activity_main是fragment的名字
再绑定视图内的navView，即导航栏，与navController，就可以使用  
```kotlin
val navController = findNavController(R.id.nav_host_fragment_activity_main)
val appBarConfiguration = AppBarConfiguration(  
    setOf(  
        R.id.navigation_my,R.id.navigation_dashboard,R.id.navigation_home,R.id.navigation_loginFragment  
    ),  
)
setupActionBarWithNavController(navController, appBarConfiguration)  
navView.setupWithNavController(navController)
```
## 1.4 fragment间的导航与传参

在navigation目录下写xml文件，在其中写导航的graph，这其中定义页面间的关系，可以直接加界面，加导航action
**重点**就是记得要**记住id好调用**

后续在fragment内需要跳转至新fragment时，直接
```
findNavController().navigate(R.id.action_navigation_q1_to_navigation_consult_result)
```
其中id是跳转的action名字

如果需要在跳转的时候进行传参操作:
1. 需要在graph内加attributions，在其中加上可以保存的参数  
```
<fragment  
    android:id="@+id/navigation_q1"  
    android:name="com.example.bottom_navigation.ui.question.Q1"  
    android:label="腰部疼痛问卷"  
    tools:layout="@layout/fragment_q1" >  
    <action  
        android:id="@+id/action_navigation_q1_to_navigation_consult_result"  
        app:destination="@id/navigation_consult_result" />  
    <argument  
        android:name="cold"  
        app:argType="integer"  
        android:defaultValue="0" />  
    <argument  
        android:name="blod"  
        app:argType="integer"  
        android:defaultValue="0" />  
    <argument  
        android:name="hott"  
        app:argType="integer"  
        android:defaultValue="0" />  
    <argument  
        android:name="kidney"  
        app:argType="integer"  
        android:defaultValue="0" />  
</fragment>
```
2. 需要跳转前写好bundle，在跳转时当参数投入进去，后续才能收到
```kotlin
val bundle = Bundle().also {  
    it.putInt("cold",cold)  
    it.putInt("blod",blod)  
    it.putInt("hott",hott)  
    it.putInt("kidney",kidney)  
}  
findNavController().navigate(R.id.action_navigation_q1_to_navigation_consult_result,bundle)
```
3. 接收时在复写onCreate函数时就进行接收操作
```kotlin
private var cold = 0  
private var blod = 0  
private var hott = 0  
private var kidney = 0
override fun onCreate(savedInstanceState: Bundle?) {  
    super.onCreate(savedInstanceState)  
    arguments?.let {  
        cold = it.getInt("cold")  
        blod = it.getInt("blod")  
        hott = it.getInt("hott")  
        kidney = it.getInt("kidney")  
    }  
}
```

# 2.正常进行返回
```kotlin
override fun onSupportNavigateUp(): Boolean {  
    return super.onSupportNavigateUp() || findNavController(R.id.nav_host_fragment_activity_main).navigateUp()  
    // 这个是返回的关键  
}
```
在最后加上这个，就能正常通过AppBar上箭头进行返回
需要在class MainActivity : AppCompatActivity()层面上复写这个函数

# 3.如何隐藏导航栏

在override fun onCreate(savedInstanceState: Bundle?)函数内
添加函数
```kotlin
navController.addOnDestinationChangedListener { _, destination, _ ->  
    if (destination.id in arrayOf(  
            R.id.navigation_my,R.id.navigation_dashboard,R.id.navigation_home  
        )) {  
        navView.visibility = View.VISIBLE  
        textview.visibility = View.VISIBLE  
        image.visibility = View.VISIBLE  
    } else {  
        navView.visibility = View.GONE  
        textview.visibility = View.GONE  
        image.visibility = View.GONE  
    }  
}
```
在destination.id加上你需要显示导航栏的页面的id
后面设置navView为View.VISIBLE
解释其中的变量，是使用binding进行的视图绑定，界面内默认设计了导航栏，标题图片，加上fragment，后续仅希望出现fragment
```kotlin
import com.example.bottom_navigation.databinding.ActivityMainBinding

private lateinit var binding: ActivityMainBinding
override fun onCreate(savedInstanceState: Bundle?) {  
    super.onCreate(savedInstanceState)
    binding = ActivityMainBinding.inflate(layoutInflater)  
	setContentView(binding.root)
	val navView: BottomNavigationView = binding.navView  
	val textview: TextView = binding.text  
	val image:ImageView = binding.imagemain
}
```
