# NotePad
## 主要功能
* **添加时间**
* **搜索功能**
## 附加功能
* **更改背景颜色**

-----------------------------------------------------------------------
### 添加时间

   首先，由于看不惯黑色背景，于是将NoteList的颜色修改与NoteEditor一致:
   在AndroidManifest.xml中添加
 ```java
   android:theme="@android:style/Theme.Holo.Light"
 ```
   
   
*  **先在noteslist_item中添加能够显示时间的TextView:**

```java
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
     >
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@android:id/text1"
    android:layout_width="match_parent"
    android:layout_height="?android:attr/listPreferredItemHeight"
    android:textAppearance="?android:attr/textAppearanceLarge"
    android:gravity="center_vertical"
    android:paddingLeft="5dip"
    android:singleLine="true"
/>
    <TextView
        android:id="@+id/date"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:textColor="#708090"
        android:textSize="20dp"
        android:singleLine="true"/>
</LinearLayout>
        
        
  ```
  *  **修改notelist.java 的PROJECTION**
  ```java
  
  private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_CREATE_DATE
    };
    
```

*  **修改dataColumns和viewID相关的值**
```java
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,NotePad.Notes.COLUMN_NAME_CREATE_DATE} ;
```
```java
int[] viewIDs = { android.R.id.text1 ,R.id.date};
```
*  **向NotePadProvider数据存储中添加时间值**
```java
SimpleDateFormat formatter   =   new   SimpleDateFormat   ("yyyy-MM-dd   HH:mm:ss");
        Date curDate =  new Date(System.currentTimeMillis());
        String   date   =   formatter.format(curDate);
        values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE,date);
           
```

*功能实现后如下图所示：*



<img src="https://github.com/increChong/NotePad-master/blob/master/ScreenShots/Screenshot_20170522-035139.png" width="50%"/>

### 搜索功能

*  **首先在list_options_menu.xml中添加一个搜索的选项：**
```java
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/menu_search"
        android:icon="@drawable/search1"
        android:title="@string/menu_add"
        android:alphabeticShortcut="s"
        android:showAsAction="always"
        />
    <!--  This is our one standard application action (creating a new note). -->
    <item android:id="@+id/menu_add"
          android:icon="@drawable/ic_menu_compose"
          android:title="@string/menu_add"
          android:alphabeticShortcut='a'
          android:showAsAction="always" />
    <!--  If there is currently data in the clipboard, this adds a PASTE menu item to the menu
          so that the user can paste in the data.. -->
    <item android:id="@+id/menu_paste"
          android:icon="@drawable/ic_menu_compose"
          android:title="@string/menu_paste"
          android:alphabeticShortcut='p' />
</menu>
```
*  **创建一个SearchView成员变量**
```java
 private SearchView searchView;
```
*  **写一个设置SearchView的方法**
```java
 private void setSearchView(Menu menu) {
        //1.找到menuItem并动态设置SearchView
        MenuItem item = menu.getItem(0);
        searchView = new SearchView(this);
        item.setActionView(searchView);

        //2.设置搜索的背景为白色
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.ICE_CREAM_SANDWICH) {
            item.collapseActionView();
        }
        searchView.setQuery("", false);
        //searchView.setBackgroundResource(R.drawable.ic_menu_edit);
        //3.设置为默认展开状态，图标在外面
        searchView.setIconifiedByDefault(true);
        searchView.setQueryHint("Search");
        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            @Override
            public boolean onQueryTextSubmit(String query) {
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.ICE_CREAM_SANDWICH) {
                    searchView.onActionViewCollapsed();
                }
                return false;
            }

            @Override
            public boolean onQueryTextChange(String newText) {
                if (!TextUtils.isEmpty(newText)) {

                    Cursor cursor = managedQuery(
                            getIntent().getData(),            // Use the default content URI for the provider.
                            PROJECTION,                       // Return the note ID and title for each note.
                            NotePad.Notes.COLUMN_NAME_TITLE + " LIKE '%" + newText + "%' ",                             // 相当于where语句
                            null,                             // No where clause, therefore no where column values.
                            NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
                    );
                    final String[] dataColumn = {NotePad.Notes.COLUMN_NAME_CREATE_DATE, NotePad.Notes.COLUMN_NAME_TITLE};

                    // The view IDs that will display the cursor columns, initialized to the TextView in
                    // noteslist_item.xml
                    int[] viewID = {R.id.date,android.R.id.text1};
                    SimpleCursorAdapter adapter1
                            = new SimpleCursorAdapter(
                            NotesList.this,                             // The Context for the ListView
                            R.layout.noteslist_item,          // Points to the XML for a list item
                            cursor,                           // The cursor to get items from
                            dataColumn,
                            viewID
                    );
                    //重新setListAdapter
                    setListAdapter(adapter1);
                } else {
                    //恢复默认setListAdapter
                    setListAdapter(adapter);
                }
                return false;
            }
        });
    }
```
*onQueryTextChange是监听搜索框的方法，根据输入文字改变SimpleCursorAdapter界面*
*  **在onCreateOptionsMenu中添加setSearchView方法**
```java
public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate menu from XML resource
        MenuInflater inflater = getMenuInflater();
        inflater.inflate(R.menu.list_options_menu, menu);
        setSearchView(menu);
        // Generate any additional actions that can be performed on the
        // overall list.  In a normal install, there are no additional
        // actions found here, but this allows other applications to extend
        // our menu with their own actions.
        Intent intent = new Intent(null, getIntent().getData());
        intent.addCategory(Intent.CATEGORY_ALTERNATIVE);
        menu.addIntentOptions(Menu.CATEGORY_ALTERNATIVE, 0, 0,
                new ComponentName(this, NotesList.class), null, intent, 0, null);

        return super.onCreateOptionsMenu(menu);
    }
```
*实现后界面如下：*



<img src="https://github.com/increChong/NotePad-master/blob/master/ScreenShots/Screenshot_20170522-035148.png" width="50%"/>
<img src="https://github.com/increChong/NotePad-master/blob/master/ScreenShots/Screenshot_20170522-035158.png" width="50%"/>

### 更改背景颜色
*  **先在NoteList的菜单栏里添加一个改变背景颜色的选项**


*在list_option_menu中添加：*


```java
<item
        android:id="@+id/bg"
        android:title="Background"
        android:alphabeticShortcut="b"
        />
```


*效果如图：*


<img src="https://github.com/increChong/NotePad-master/blob/master/ScreenShots/Screenshot_20170524-132652.png" width="50%"/>

*  **创建一个用来显示颜色选择的布局color.xml**

```java
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:layout_width="match_parent"
    android:layout_height="100dp">
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="#E1FFFF"
        android:id="@+id/LightCyan"
        android:clickable="true"
        android:layout_weight="1"/>
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="#FFC0CB"
        android:id="@+id/Pink"
        android:layout_weight="1"/>

    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="#87CEFA"
        android:id="@+id/LightSkyBlue"
        android:layout_weight="1"/>
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="#F8F8FF"
        android:id="@+id/GhostWhite"
        android:layout_weight="1"/>
</LinearLayout>
```

*  **使用SharedPreferences来保存之前设置的颜色，在下次启动应用时设置**


*创建一个SharedPreferences变量和存储颜色的变量：*


```java
    private SharedPreferences bg;
    private String preferencescolor;
```


*使用SharedPreferences保存颜色的方法：*



```java
private void putColor(String color){
        SharedPreferences preferences = getSharedPreferences("preferences", Context.MODE_PRIVATE);
        SharedPreferences.Editor editor = preferences.edit();
        editor.putString("color", color);
        editor.commit();
    }
```



*在onCreate方法中设置，如果之前有设置过颜色，启动时应用，没有则使用默认颜色：*



```java
bg = getSharedPreferences("preferences", Context.MODE_PRIVATE);
        preferencescolor=bg.getString("color", "");
        //设置Listview的背景色
        if(preferencescolor.equals("")){
            getListView().setBackgroundColor(Color.parseColor("#FFFFFF"));
        }
        else {
            getListView().setBackgroundColor(Color.parseColor(preferencescolor));
        }
```


*  **在onOptionItemSelected方法中设置Background的响应事件**

```java
public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
        case R.id.menu_add:
          /*
           * Launches a new Activity using an Intent. The intent filter for the Activity
           * has to have action ACTION_INSERT. No category is set, so DEFAULT is assumed.
           * In effect, this starts the NoteEditor Activity in NotePad.
           */
           startActivity(new Intent(Intent.ACTION_INSERT, getIntent().getData()));
           return true;
        case R.id.menu_paste:
          /*
           * Launches a new Activity using an Intent. The intent filter for the Activity
           * has to have action ACTION_PASTE. No category is set, so DEFAULT is assumed.
           * In effect, this starts the NoteEditor Activity in NotePad.
           */
          startActivity(new Intent(Intent.ACTION_PASTE, getIntent().getData()));
            return true;
            //Background的点击事件
            case R.id.bg:
                final AlertDialog alertDialog = new AlertDialog.Builder(NotesList.this).create();
                alertDialog.show();
                Window window = alertDialog.getWindow();
                window.setContentView(R.layout.color);

                TextView color_LightCyan = (TextView)alertDialog.getWindow().findViewById(R.id.LightCyan);
                TextView color_Pink = (TextView)alertDialog.getWindow().findViewById(R.id.Pink);
                TextView color_LightSkyBlue = (TextView)alertDialog.getWindow().findViewById(R.id.LightSkyBlue);
                TextView color_GhostWhite = (TextView)alertDialog.getWindow().findViewById(R.id.GhostWhite);

                color_LightCyan.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        preferencescolor="#E1FFFF";
                        getListView().setBackgroundColor(Color.parseColor(preferencescolor));
                        putColor(preferencescolor);
                        alertDialog.dismiss();
                    }
                });

                color_Pink.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        preferencescolor="#FFC0CB";
                        getListView().setBackgroundColor(Color.parseColor(preferencescolor));
                        putColor(preferencescolor);
                        alertDialog.dismiss();
                    }
                });

                color_LightSkyBlue.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        preferencescolor="#87CEFA";
                        getListView().setBackgroundColor(Color.parseColor(preferencescolor));
                        putColor(preferencescolor);
                        alertDialog.dismiss();
                    }
                });

                color_GhostWhite.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        preferencescolor="#F8F8FF";
                        getListView().setBackgroundColor(Color.parseColor(preferencescolor));
                        putColor(preferencescolor);
                        alertDialog.dismiss();
                    }
                });
          return true;
        default:
            return super.onOptionsItemSelected(item);
        }
    }
```


*alertDialog是用来显示刚才创建color.xml;然后给每个颜色设置点击事件，将色值传给preferencescolor并设置为背景*


*实现后如下图：*


<img src="https://github.com/increChong/NotePad-master/blob/master/ScreenShots/Screenshot_20170524-132705.png" width="50%"/>
<img src="https://github.com/increChong/NotePad-master/blob/master/ScreenShots/Screenshot_20170524-132716.png" width="50%"/>
