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
在NoteEditor.java中的updateNote()方法修改插入部分代码，存储的是创建时间暨第一次修改时间
```
//修改时间
        Long now = Long.valueOf(System.currentTimeMillis());
        Date date = new Date(now);
        SimpleDateFormat format = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");
        String dateTime = format.format(date);
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateTime);
```
