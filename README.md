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
4、修改NotesList.java的onOptionsItemSelected()方法，在switch中插入关于搜索的case语句  
```
      case R.id.menu_search:
          Intent intent = new Intent();
          intent.setClass(NotesList.this,NoteSearch.class);
          NotesList.this.startActivity(intent);
          return true;
```
5、在AndroidManifest.xml注册NoteSearch  
```
<!--添加搜索activity-->
    <activity
        android:name="NoteSearch"
        android:label="@string/title_notes_search">
    </activity>
```
设置完毕，搜索功能完成  
搜索显示截图：  


### 设置背景图片  
1、在editor_option_menu.xml中设置关于设置背景图片的项目  
```
<item android:id="@+id/image_ground"
        android:title="设置背景图片" />
```
2、在NoteEditor.java中的onOptionsItemSelected()方法的switch语句中添加  
```
case R.id.image_ground:
                gallery();
                break;
```
3、在NoteEditor中添加全局变量  
```
    private static final int REQUEST_CODE_GALLERY = 0x10;// 图库选取图片标识请求码
    private static final int CROP_PHOTO = 0x12;// 裁剪图片标识请求码
    private static final int STORAGE_PERMISSION = 0x20;// 动态申请存储权限标识
    private File imageFile = null;// 声明File对象
    private Uri imageUri = null;// 裁剪后的图片uri
```
4、获取图片url  
在NoteEditor中添加代码
```
 /**
     * 图库选择图片
     */
    private void gallery() {

        Intent intent = new Intent(Intent.ACTION_PICK, MediaStore.Images.Media.EXTERNAL_CONTENT_URI);
        // 以startActivityForResult的方式启动一个activity用来获取返回的结果
        startActivityForResult(intent, REQUEST_CODE_GALLERY);
    }

    /**
     * 接收#startActivityForResult(Intent, int)调用的结果
     * @param requestCode 请求码 识别这个结果来自谁
     * @param resultCode    结果码
     * @param data
     */
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
//        super.onActivityResult(requestCode, resultCode, data);
        if (resultCode == RESULT_OK){// 操作成功了

            switch (requestCode){

                case REQUEST_CODE_GALLERY:// 图库选择图片

                    Uri uri = data.getData();// 获取图片的uri

                    Intent intent_gallery_crop = new Intent("com.android.camera.action.CROP");
                    intent_gallery_crop.setDataAndType(uri, "image/*");

                    // 设置裁剪
                    intent_gallery_crop.putExtra("crop", "true");
                    intent_gallery_crop.putExtra("scale", true);
                    // aspectX aspectY 是宽高的比例
                    intent_gallery_crop.putExtra("aspectX", 1);
                    intent_gallery_crop.putExtra("aspectY", 1);
                    // outputX outputY 是裁剪图片宽高
                    intent_gallery_crop.putExtra("outputX", 400);
                    intent_gallery_crop.putExtra("outputY", 400);

                    intent_gallery_crop.putExtra("return-data", false);

                    // 创建文件保存裁剪的图片
                    createImageFile();
                    imageUri = Uri.fromFile(imageFile);

                    if (imageUri != null){
                        intent_gallery_crop.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
                        intent_gallery_crop.putExtra("outputFormat", Bitmap.CompressFormat.JPEG.toString());
                    }

                    startActivityForResult(intent_gallery_crop, CROP_PHOTO);

                    break;

                case CROP_PHOTO:// 裁剪图片

                    try{

                        if (imageUri != null){
                            displayImage(imageUri);
                        }

                    }catch (Exception e){
                        e.printStackTrace();
                    }

                    break;

            }

        }
    }

    /**
     * 创建File保存图片
     */
    private void createImageFile() {

        try{

            if (imageFile != null && imageFile.exists()){
                imageFile.delete();
            }
            // 新建文件
            imageFile = new File(Environment.getExternalStorageDirectory(),
                    System.currentTimeMillis() + "galleryDemo.jpg");
        }catch (Exception e){
            e.printStackTrace();
        }

    }
```
5、显示图片  
先在gadle(app)中将minsdk设置为16  
在NoteEditor中添加代码：  
```
private void displayImage(Uri imageUri) {
        Drawable drawable= Drawable.createFromPath(imageUri.toString());
        getWindow().getDecorView().setBackground(drawable);  
    }

```
6、在NoteEditor的PROJECTION中添加一行NotePad.Notes.COLUMN_NAME_BACK_IMAGE  
```
private static final String[] PROJECTION =
        new String[] {
            NotePad.Notes._ID,
            NotePad.Notes.COLUMN_NAME_TITLE,
            NotePad.Notes.COLUMN_NAME_NOTE,
            NotePad.Notes.COLUMN_NAME_BACK_IMAGE
    };
```
7、在NoteEditor的updateNote()方法中添加代码，记录url  
```
values.put(NotePad.Notes.COLUMN_NAME_BACK_IMAGE, imageUri.toString());
```
8、在NotePad中添加  
```
public static final String COLUMN_NAME_BACK_IMAGE= "back_image";
```
9、在NotePadProvider中的sql表格添加一行+ NotePad.Notes.COLUMN_NAME_BACK_IMAGE + " TEXT"   
```
@Override
       public void onCreate(SQLiteDatabase db) {
           db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                   + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                   + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_BACK_IMAGE + " TEXT"
                   + ");");
       }
```
10、在NotePadProvider中的insert()添加  
```
if (values.containsKey(NotePad.Notes.COLUMN_NAME_BACK_IMAGE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_BACK_IMAGE, "");
        }
```
设置完毕，设置背景图片功能完成  
设置图片背景截图（显示图片方法不同，背景可能会为黑色） 



### 设置背景颜色
1、在NotePad中添加：
```
public static final String COLUMN_NAME_BACK_COLOR = "color";
        public static final int DEFAULT_COLOR = 0; //颜色对应的数值
        public static final int YELLOW_COLOR = 1;
        public static final int BLUE_COLOR = 2;
        public static final int RED_COLOR = 4;
        public static final int BLACK_COLOR=5;
```
2、在editor_options_menu.xml中添加项目及子项目
```
<item android:title="设置背景颜色"
        android:id="@+id/back_color">
        <menu>
            <item android:id="@+id/black"
                android:title="Black" />
            <item android:id="@+id/red"
                android:title="Pink" />
            <item android:id="@+id/yellow"
                android:title="Yellow" />
            <item android:id="@+id/blue"
                android:title="Blue"/>
        </menu>
    </item>
```
3、在NoteEditor中设置全局变量backcolor默认值为0
```
  private int backcolor=NotePad.Notes.DEFAULT_COLOR;
```
4、在NoteEditor中的onOptionsItemSelected()添加：
```
case R.id.back_color:
            case R.id.red:
                changeColor(NotePad.Notes.YELLOW_COLOR);
                break;
            case R.id.black:
                changeColor(NotePad.Notes.BLUE_COLOR);
                break;
            case R.id.blue:
                changeColor(NotePad.Notes.BLACK_COLOR);
                break;
            case R.id.yellow:
                changeColor(NotePad.Notes.DEFAULT_COLOR);
                break;
```
5、在NoteEdittor中写入代码响应选项：
```
private final void changeColor(int color) {
        switch(color) {
            case NotePad.Notes.YELLOW_COLOR:
                getWindow().getDecorView().setBackgroundColor(getResources().getColor(R.color.yellow));
                backcolor=NotePad.Notes.YELLOW_COLOR;
                break;
            case NotePad.Notes.BLUE_COLOR:
                getWindow().getDecorView().setBackgroundColor(getResources().getColor(R.color.blue));
                backcolor=NotePad.Notes.BLUE_COLOR;
                break;
            case NotePad.Notes.RED_COLOR:
                getWindow().getDecorView().setBackgroundColor(getResources().getColor(R.color.red));
                backcolor=NotePad.Notes.RED_COLOR;
                break;
            case NotePad.Notes.BLACK_COLOR:
                getWindow().getDecorView().setBackgroundColor(getResources().getColor(R.color.black));
                backcolor=NotePad.Notes.BLACK_COLOR;
                break;
            default:
                getWindow().getDecorView().setBackgroundColor(getResources().getColor(R.color.white));
                backcolor=NotePad.Notes.DEFAULT_COLOR;
                break;
        }
    }
```
6、在NoteEditor的PROJECTION中添加一行NotePad.Notes.COLUMN_NAME_BACK_COLOR
```
private static final String[] PROJECTION =
        new String[] {
            NotePad.Notes._ID,
            NotePad.Notes.COLUMN_NAME_TITLE,
            NotePad.Notes.COLUMN_NAME_NOTE,
            NotePad.Notes.COLUMN_NAME_BACK_IMAGE,
            NotePad.Notes.COLUMN_NAME_BACK_COLOR
    };
```
7、在NoteEditor的updateNote()方法中添加代码，记录颜色编码  
```
values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, backcolor);
```
8、在NoteEditor的OnReSume()方法中添加代码,初始化背景颜色  
```
int color = mCursor.getInt(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
                switch(color) {
                    case NotePad.Notes.YELLOW_COLOR:
                        getWindow().getDecorView().setBackgroundColor(getResources().getColor(R.color.yellow));
                        backcolor=NotePad.Notes.YELLOW_COLOR;
                        break;
                    case NotePad.Notes.BLUE_COLOR:
                        getWindow().getDecorView().setBackgroundColor(getResources().getColor(R.color.blue));
                        backcolor=NotePad.Notes.BLUE_COLOR;
                        break;
                    case NotePad.Notes.RED_COLOR:
                        getWindow().getDecorView().setBackgroundColor(getResources().getColor(R.color.red));
                        backcolor=NotePad.Notes.RED_COLOR;
                        break;
                    case NotePad.Notes.BLACK_COLOR:
                        getWindow().getDecorView().setBackgroundColor(getResources().getColor(R.color.black));
                        backcolor=NotePad.Notes.BLACK_COLOR;
                        break;
                    default:
                        getWindow().getDecorView().setBackgroundColor(getResources().getColor(R.color.white));
                        backcolor=NotePad.Notes.DEFAULT_COLOR;
                        break;
                }
```
9、在NotePadProvider中的sql表格添加一行+ NotePad.Notes.COLUMN_NAME_BACK_IMAGE + " TEXT"   
```
@Override
       public void onCreate(SQLiteDatabase db) {
           db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                   + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                   + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_BACK_IMAGE + " TEXT"
                   + ");");
       }
```
10、在NotePadProvider中的insert()添加  
```
if (values.containsKey(NotePad.Notes.COLUMN_NAME_BACK_COLOR) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, NotePad.Notes.DEFAULT_COLOR);
        }
```
11、修改NoteList  
```
private static final String[] PROJECTION = new String[] {
           NotePad.Notes._ID, // 0
           NotePad.Notes.COLUMN_NAME_TITLE, // 1
           //扩展 显示时间 颜色
           NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
           NotePad.Notes.COLUMN_NAME_BACK_COLOR,
   };
```
设置完成，设置背景颜色功能完成  
设置背景颜色截图：  


### 根据时间排序
