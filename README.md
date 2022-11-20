# mid_test

![image](https://user-images.githubusercontent.com/77438764/202893700-507c5bd0-2b59-477d-a60d-97fdb030d30b.png)

**基本功能一：时间戳显示**


代码部分：

格式化
```
   Long now = Long.valueOf(System.currentTimeMillis());
Date date = new Date(now);
SimpleDateFormat format = new SimpleDateFormat("yy.MM.dd HH:mm:ss");
String dateTime = format.format(date);

```
  
  
 显示时间戳
 ```
   <TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:textColor="@android:color/holo_purple"
        android:textSize="20sp"
        />
        
```
  
 NodeList修改部分：
 
 
```
   private static final String TAG = "NotesList";
private static final String[] PROJECTION = new String[] {
        NotePad.Notes._ID, // 0
        NotePad.Notes.COLUMN_NAME_TITLE, // 1
        NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//时间戳
};
```


  
  
  截图
  
  
  
  ![image](https://user-images.githubusercontent.com/77438764/202893947-fa2f7b55-980c-493b-88b6-aebce3eb863e.png)
  
  
  
  
**基本功能二：搜索**

```
 <!--菜单中增加一个搜索项-->
    <item
        android:id="@+id/menu_search"
        android:title="@string/menu_search"
        android:icon="@android:drawable/ic_search_category_default"
        android:showAsAction="always">
    </item>
```
  
  添加搜索
  
  
```
              case R.id.menu_search:
                Intent intent = new Intent();
                intent.setClass(NotesList.this,NoteSearch.class);
                NotesList.this.startActivity(intent);
                return true;
```
         public class NoteSearch extends ListActivity  implements SearchView.OnQueryTextListener {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search);
        Intent intent = getIntent(); 
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        SearchView searchview = (SearchView)findViewById(R.id.search_view); 
        searchview.setOnQueryTextListener(NoteSearch.this);  
    }

    @Override
    public boolean onQueryTextSubmit(String query) {
        return false;
    }

    @Override
    //Text改变时执行的内容
    public boolean onQueryTextChange(String newText) {

        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";

        //实现模糊查询
        String[] selectionArgs = { "%"+newText+"%" };
        Cursor cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                selection,
                selectionArgs,
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );

        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
        int[] viewIDs = { android.R.id.text1 , R.id.text2 };
        SimpleCursorAdapter adapter = new SimpleCursorAdapter(
                this,                  
                R.layout.noteslist_item,        
                cursor,                        
                dataColumns,                    
                viewIDs                         
        );
        setListAdapter(adapter);
        return true;
    }

    @Override
    protected void onListItemClick(ListView l, View v, int position, long id) {

        // Constructs a new URI from the incoming URI and the row ID
        Uri uri = ContentUris.withAppendedId(getIntent().getData(), id);

        // Gets the action from the incoming Intent
        String action = getIntent().getAction();

        // Handles requests for note data
        if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {

            // Sets the result to return to the component that called this Activity. The
            // result contains the new URI
            setResult(RESULT_OK, new Intent().setData(uri));
        } else {

            // Sends out an Intent to start an Activity that can handle ACTION_EDIT. The
            // Intent's data is the note ID URI. The effect is to call NoteEdit.
            startActivity(new Intent(Intent.ACTION_EDIT, uri));
        }
    }
}


  ![image](https://user-images.githubusercontent.com/77438764/202895130-86975b5d-35ad-44bb-90be-0c3b93c68601.png)
  
  
  
![image](https://user-images.githubusercontent.com/77438764/202895137-85f509bf-b7c8-425a-80b0-5ab74a6de2f6.png)




**额外内容**
**额外功能一：排序**


根据创建时间排序


```
case R.id.menu_sort1:
    cursor = managedQuery(
            getIntent().getData(),
            PROJECTION,
            null,
            null,
            NotePad.Notes._ID
    );
    adapter = new MyCursorAdapter(
            this,
            R.layout.noteslist_item,
            cursor,
            dataColumns,
            viewIDs
    );
    setListAdapter(adapter);
    return true;

```

根据修改时间排序

```
case R.id.menu_sort2:
    cursor = managedQuery(
            getIntent().getData(),
            PROJECTION,
            null,
            null,
            NotePad.Notes.DEFAULT_SORT_ORDER
    );

    adapter = new MyCursorAdapter(
            this,
            R.layout.noteslist_item,
            cursor,
            dataColumns,
            viewIDs
    );
    setListAdapter(adapter);
    return true;
```


  ![image](https://user-images.githubusercontent.com/77438764/202895281-28b58a92-14e1-444e-9786-29cf14bd80d7.png)
  
  
  
![image](https://user-images.githubusercontent.com/77438764/202895292-cc882a75-306b-456c-84ff-3001f3d73969.png)



**额外内容二：UI美化**

修改NotePad:


```
public static final int DEFAULT_COLOR = 0; //白
public static final int YELLOW_COLOR = 1; //黄
public static final int BLUE_COLOR = 2; //蓝
public static final int GREEN_COLOR = 3; //绿
public static final int RED_COLOR = 4; //红

```


修改NotePadProvider:


```
public class MyCursorAdapter extends SimpleCursorAdapter {
    public MyCursorAdapter(Context context, int layout, Cursor c,
                           String[] from, int[] to) {
        super(context, layout, c, from, to);
    }
    @Override
    public void bindView(View view, Context context, Cursor cursor){
        super.bindView(view, context, cursor);
        //从数据库中读取的cursor中获取笔记列表对应的颜色数据，并设置笔记颜色
        int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
        switch (x){
            case NotePad.Notes.DEFAULT_COLOR:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
            case NotePad.Notes.YELLOW_COLOR:
                view.setBackgroundColor(Color.rgb(247, 216, 133));
                break;
            case NotePad.Notes.BLUE_COLOR:
                view.setBackgroundColor(Color.rgb(165, 202, 237));
                break;
            case NotePad.Notes.GREEN_COLOR:
                view.setBackgroundColor(Color.rgb(161, 214, 174));
                break;
            case NotePad.Notes.RED_COLOR:
                view.setBackgroundColor(Color.rgb(244, 149, 133));
                break;
            default:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
        }
    }
}
```

在noteList当中添加颜色的显示：


```
private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //扩展 显示时间 颜色
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
    };
```

在NoteEditor 当中的代码：
```
  private static final String[] PROJECTION =
        new String[] {
            NotePad.Notes._ID,
            NotePad.Notes.COLUMN_NAME_TITLE,
            NotePad.Notes.COLUMN_NAME_NOTE,
            NotePad.Notes.COLUMN_NAME_BACK_COLOR
    };
```

onResume()中代码：

```
int x = mCursor.getInt(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
switch (x){
    case NotePad.Notes.DEFAULT_COLOR:
        mText.setBackgroundColor(Color.rgb(255, 255, 255));
        break;
    case NotePad.Notes.YELLOW_COLOR:
        mText.setBackgroundColor(Color.rgb(247, 216, 133));
        break;
    case NotePad.Notes.BLUE_COLOR:
        mText.setBackgroundColor(Color.rgb(165, 202, 237));
        break;
    case NotePad.Notes.GREEN_COLOR:
        mText.setBackgroundColor(Color.rgb(161, 214, 174));
        break;
    case NotePad.Notes.RED_COLOR:
        mText.setBackgroundColor(Color.rgb(244, 149, 133));
        break;
    default:
        mText.setBackgroundColor(Color.rgb(255, 255, 255));
        break;
}

```

字体大小：

```
<item
    android:id="@+id/item1"
    android:title="字体大小">
    <menu>
        <item
            android:id="@+id/small"
            android:title="小">
        </item>
        <item
            android:id="@+id/middle"
            android:title="中">
        </item>
        <item
            android:id="@+id/big"
            android:title="大">
        </item>

    </menu>
</item>

```

字体颜色：

```
<item android:id="@+id/item3"
    android:title="字体颜色">
    <menu>
        <item
            android:title="红"
            android:id="@+id/red">

        </item>
        <item android:title="黑"
            android:id="@+id/black">

        </item>
    </menu>
</item>

```
功能截图：

![image](https://user-images.githubusercontent.com/77438764/202895732-4ef18b9d-f3d3-478a-9f30-550e24beb358.png)


![image](https://user-images.githubusercontent.com/77438764/202896339-d8e9582e-cbef-4526-aeda-6e4afb1d0094.png)


![image](https://user-images.githubusercontent.com/77438764/202896347-0d44f7ae-a0a1-477e-9b04-dabdce02bc08.png)



![image](https://user-images.githubusercontent.com/77438764/202896372-4c17f3a7-a92a-4d12-bac3-8b3fa442bae8.png)



