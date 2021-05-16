# VIEW BINDING

[**Setup**](#Setup)

[**Activity usage**](#Activity-usage)

[**Fragment usage**](#Fragment-usage)

&nbsp;&nbsp;&nbsp;&nbsp;[Using inflate method](#Using-inflate-method)

&nbsp;&nbsp;&nbsp;&nbsp;[Using bind method](#Using-bind-method)

[**RecyclerView usage**](#RecyclerView-usage)

[**Include tag usage**](#Include-tag-usage)

&nbsp;&nbsp;&nbsp;&nbsp;[Using without the merge tag](#Using-without-the-merge-tag)

&nbsp;&nbsp;&nbsp;&nbsp;[Using with the merge tag](#Using-with-the-merge-tag)

<hr/>

## Setup

```guava
android {
    ...
    buildFeatures {
        viewBinding true
    }
}
```

To ignore per a layout file, use the following tag:

```xml
<LinearLayout
        tools:viewBindingIgnore="true" >
</LinearLayout>
```

## Activity usage

```kotlin
private var _binding: ActivityMainBinding? = null
private val binding get() = _binding!!

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    _binding = ActivityMainBinding.inflate(layoutInflater)
    val view = binding.root
    setContentView(view)
}

override fun onDestroy() {
    _binding = null
    super.onDestroy()
}
```

## Fragment usage

### Using inflate method

```kotlin
private var _binding: ResultProfileBinding? = null
private val binding get() = _binding!!

override fun onCreateView(
    inflater: LayoutInflater,
    container: ViewGroup?,
    savedInstanceState: Bundle?
): View? {
    _binding = ResultProfileBinding.inflate(inflater, container, false)
    val view = binding.root
    return view
}

override fun onDestroyView() {
    _binding = null
    super.onDestroyView()
}
```

### Using bind method

```kotlin
class MainFragment : Fragment(R.layout.fragment_main) {

    private var _binding: MainBinding? = null
    private val binding get() = _binding!!

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        _binding = MainBinding.bind(view)
        // binding.textView.text = ...
    }

    override fun onDestroyView() {
        _binding = null
        super.onDestroyView()
    }
}
```

## RecyclerView usage

```kotlin
...
override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): CustomViewHolder {
    val itemBinding = ActivityMainBinding.inflate(LayoutInflater.from(parent.context), parent, false)
    return CustomViewHolder(itemBinding)
}
...

class CustomViewHolder(private val itemBinding: ItemBinding) : RecyclerView.ViewHolder(itemBinding.root) {
    fun bind(itemModel: Any) {
        itemBinding.textView.text = itemModel.name
    }
}
```

## Include tag usage

### Using without the merge tag

```xml
// Include file
<androidx.constraintlayout.widget.ConstraintLayout>

    <androidx.appcompat.widget.Toolbar
        android:id="@+id/toolbar" />
    
</androidx.constraintlayout.widget.ConstraintLayout>
```

```xml
// Activity/Fragment layout file
<androidx.constraintlayout.widget.ConstraintLayout>

    <include
        android:id="@+id/appbar"
        layout="@layout/app_bar" />
    
</androidx.constraintlayout.widget.ConstraintLayout>
```

```kotlin
binding.appbar.toolbar
```

### Using with the merge tag

```xml
// Include file
<merge>
    
    <TextView
        android:id="@+id/tvPlaceholder" />
    
</merge>
```

```xml
// Activity/Fragment layout file
<androidx.constraintlayout.widget.ConstraintLayout>

    <include layout="@layout/placeholder" />
    
</androidx.constraintlayout.widget.ConstraintLayout>
```

In this case, we get an auto-generated class `PlaceholderBinding` which corresponds to placeholder.xml (our \<merge> layout file)

```kotlin
private var _placeholderBinding: PlaceholderBinding? = null
private val placeholderBinding get() = _placeholderBinding!!

override fun onCreateView(
    inflater: LayoutInflater,
    container: ViewGroup?,
    savedInstanceState: Bundle?
): View? {
    _binding = ResultProfileBinding.inflate(inflater, container, false)
    val view = binding.root
   
    // We have to use the bind method and pass the root view
    _placeholderBinding = PlaceholderBinding.bind(view)
    //placeholderBinding.tvPlaceholder.text = ..

    return view
}

override fun onDestroyView() {
    _placeholderBinding = null
    super.onDestroyView()
}
```



