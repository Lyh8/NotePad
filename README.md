# 修改时间戳


**1.修改res/layout下的noteslist_item.xml**

  原先只有一个TextView, 用来显示笔记的标题，要想显示时间戳，必须再加入一个TextView,并将布局修改为RelativeLayout，修改后的noteslist_item.xml的代码为:

`

```
<RelativeLayout android:layout_height="match_parent"
    android:layout_width="match_parent"
    xmlns:android="http://schemas.android.com/apk/res/android">
    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:textSize="30dp"
        android:paddingLeft="5dip"
        android:singleLine="true"
        />
    <TextView
        android:id="@+id/time"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="15dp"
        android:paddingLeft="5dip"
        android:paddingTop="@android:dimen/app_icon_size"
        android:singleLine="true"
        android:gravity="center_vertical"
       />

</RelativeLayout>
```

`

**2.修改时间戳类型**

在新建笔记NotePadProvider中的insert方法中进行修改，修改的代码如下:

`

```
Long now = Long.valueOf(System.currentTimeMillis());
//将时间格式转为yyyy/MM/dd hh:mm a的形式
Date date = new Date(now);
SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy/MM/dd hh:mm a");
String dateFormat = simpleDateFormat.format(date);

// If the values map doesn't contain the creation date, sets the value to the current time.
if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
    values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, dateFormat );
}

// If the values map doesn't contain the modification date, sets the value to the current
// time.
if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
    values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateFormat );
}
```

`

修改笔记在NoteEditor中的updateNote方法中进行修改，修改的代码如下:

`

```
Long now = Long.valueOf(System.currentTimeMillis());
Date date = new Date(now);
SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy/MM/dd hh:mm a");
String dateFormat = simpleDateFormat.format(date);
values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateFormat );
```

`

**3.NoteList的修改**

在NotePadProvider中的onCreate(SQLiteDatabase db)可以看到数据表的创建是含有创建时间和修改时间的列的：

`

```
public void onCreate(SQLiteDatabase db) {
    db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
            + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
            + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
            + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
            + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
            + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER"
            + ");");
}
```

`

只是在NotesList中的PROJECTION没有投影出来，于是我们新添加对修改时间的列的投影：

`

```
private static final String[] PROJECTION = new String[] {
        NotePad.Notes._ID, // 0
        NotePad.Notes.COLUMN_NAME_TITLE, // 1
        NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
};
```

`

PROJECTION只是定义了需要被取出来的数据列，而之后用Cursor进行数据库查询，再之后用Adapter进行装填。
源代码Cursor不用变化，我们需要将显示列dataColumns和他们的viewIDs加入修改时间这一属性

`

```
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE , NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;

// The view IDs that will display the cursor columns, initialized to the TextView in
// noteslist_item.xmls
int[] viewIDs = { android.R.id.text1 ,R.id.time };//添加修改时间，利用第一步里TextView的id
```

`

**4.显示效果**

![看不见图片在screenshot下可看到](screenshot\sreenshot.jpg)
