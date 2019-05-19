## MyNotePad
A simple Notepad provides basic functionality 
## 一、笔记的功能介绍    
### 1、原有功能  
新建笔记  
编辑笔记  
保存笔记  
删除笔记  
编辑笔记标题  
点击标题打开、删除笔记  
### 2、必须实现的两个功能  
笔记修改时间的显示  
笔记内容的搜索功能  
### 3、扩展的功能  
设置背景图片  
设置背景颜色  
按时间排序  
多选批量删除笔记  
## 二、功能的实现    
  
### 修改笔记的时间的显示 
1、修改notelist_item，添加一个LinearLayout和TextView
```
<LinearLayout  xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
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
        android:id="@+id/text1_time"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceSmall"
        android:paddingLeft="5dip"
        android:textColor="@color/black"/>
</LinearLayout>
```
2、在NoteList.java的PROJECTION中添加一行记录修改时间的数据代码NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE  
```
private static final String[] PROJECTION = new String[] {
           NotePad.Notes._ID, // 0
           NotePad.Notes.COLUMN_NAME_TITLE, // 
           NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
   };
```
3、在NoteList.java修改dataColumns和viewIDs用以获得显示修改时间
```
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE } ;

    // The view IDs that will display the cursor columns, initialized to the TextView in
    // noteslist_item.xml
    int[] viewIDs = { android.R.id.text1 ,R.id.text1_time};
```
4、获取当前时间并格式化存入数据库:   
在NotePadProvider.java中的insert()方法修改插入部分代码，存储的是创建时间暨第一次修改时间
```
Long now = Long.valueOf(System.currentTimeMillis());
        Date date = new Date(now);
        SimpleDateFormat format = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");
        String dateTime = format.format(date);
```
在NoteEditor.java中的updateNote()方法修改插入部分代码，存储的最新修改时间
```
//修改时间
        Long now = Long.valueOf(System.currentTimeMillis());
        Date date = new Date(now);
        SimpleDateFormat format = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");
        String dateTime = format.format(date);
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateTime);
```
设置完毕，完成时间的显示  
时间显示截图：  

  
### 根据标题搜索笔记
1、在list_options_menu.xml中添加搜索项:  
```
<item
        android:id="@+id/menu_search"
        android:title="@string/menu_search"
        android:icon="@android:drawable/ic_search_category_default"
        android:showAsAction="always">
    </item>
    ```
2、新建一个用于显示搜索页面的LinearLayout布局文件note_search_list.xml： 
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">

    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        android:queryHint="输入搜索内容..."
        android:layout_alignParentTop="true">
    </SearchView>

    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </ListView>
</LinearLayout>
```
3、新建一个activity名为NoteSearch，继承ListActivity类SearchView.OnQueryTextListener接口  
```
package com.example.android.notepad;

import android.app.ListActivity;
import android.widget.SearchView;
import android.app.ListActivity;
import android.content.ContentUris;
import android.content.Intent;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;
import android.widget.ListView;
import android.widget.SearchView;
import android.widget.SimpleCursorAdapter;

public class NoteSearch extends ListActivity implements SearchView.OnQueryTextListener {
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //扩展 显示时间 颜色
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
         //   NotePad.Notes.COLUMN_NAME_BACK_COLOR
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search_list);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        SearchView searchview = (SearchView)findViewById(R.id.search_view);
        searchview.setOnQueryTextListener(NoteSearch.this);  //为查询文本框注册监听器
    }

    @Override
    public boolean onQueryTextSubmit(String query) {
        return false;
    }

    @Override
    public boolean onQueryTextChange(String newText) {

        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";

        String[] selectionArgs = { "%"+newText+"%" };

        Cursor cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                selection,                        // 条件左边
                selectionArgs,                    // 条件右边
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );

        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
        int[] viewIDs = { android.R.id.text1 , R.id.text1_time };

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

```
```
```
