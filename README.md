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
