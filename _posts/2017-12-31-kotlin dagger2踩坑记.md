# kotlin dagger2踩坑记

1. 配置

   ```groovy
   dependencies{
       compile 'com.google.dagger:dagger-android:2.14.1'
       compile 'com.google.dagger:dagger-android-support:2.14.1'
       kapt 'com.google.dagger:dagger-compiler:2.14.1'
   }
   ```

   ​

2. @Named注解的使用

   ListType.kt

   ```kotlin
   object ListType {
       const val VERTICAL = "vertical"
       const val HORIZONTAL = "horizontal"
       const val GRID = "grid"
   }
   ```

   moduler

   ```kotlin
   @Module
   class LayoutManagerModuler(val context: Context) {

       @Named(ListType.VERTICAL)
       @Provides
       fun provideLayoutManagerV(): RecyclerView.LayoutManager = LinearLayoutManager(context, LinearLayoutManager.VERTICAL, false)


       @Named(ListType.HORIZONTAL)
       @Provides
       fun provideLayoutManagerH(): RecyclerView.LayoutManager = LinearLayoutManager(context, LinearLayoutManager.HORIZONTAL, false)

       @Named(ListType.GRID)
       @Provides
       fun provideLayoutManagerGrid(): RecyclerView.LayoutManager = GridLayoutManager(context, 4)

       @Provides
       fun provideResourse() = context.resources

   }
   ```

   在filed中使用named

   ```kotlin
   class MainActivity : AppCompatActivity() {

       @field:[Inject Named(ListType.VERTICAL)]
       lateinit var layoutV: RecyclerView.LayoutManager

       @field:[Inject Named(ListType.HORIZONTAL)]
       lateinit var layoutH: RecyclerView.LayoutManager


       @field:[Inject Named(ListType.GRID)]
       lateinit var layoutGrid: RecyclerView.LayoutManager

       @Inject
       lateinit var resource:Resources

       @Inject
       lateinit var listV:ListV

       override fun onCreate(savedInstanceState: Bundle?) {
           super.onCreate(savedInstanceState)
           DaggerMainComp.builder().layoutManagerModuler(LayoutManagerModuler(this)).build().inject(this)

           Log.i("test", "di result ${resource.toString()}")
           setContentView(R.layout.activity_main)
       }
   }
   ```

   在constructor中使用Named

   ```kotlin
   class ListV @Inject public constructor(@Named(ListType.VERTICAL) layoutmanager:RecyclerView.LayoutManager)
   ```

   ​

   ​