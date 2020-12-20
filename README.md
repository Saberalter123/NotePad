# 期中实验项目NotePad
 This is an AndroidStudio rebuild of google SDK sample NotePad
# NotePad主要目录结构
 ------------------
 ![项目结构](https://github.com/Saberalter123/NotePad/image/img "项目结构图")
 
# 笔记显示时间戳功能
-----------------
## 通过对以下NotePad这段源代码的分析，不难发现NotePad的数据表里已经包括条目的创建时间和最晚更新时间。这样我们只需要修改条目的样式，并把最晚更新时间显示到对应的位置即可。
* 在Notelist中找到取出数据库中数据，并将其投影在屏幕中的方法，发现要显示时间戳，必须先创建一个显示的文本框，于是修改notelist_item.xml，添加的代码如下：
    <TextView
            android:id="@+id/text3"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textColor="#00BFFF"/>
* 在NotePadProvider中先增加需要显示的最晚修改时间的映射进PROJECTION中。修改后代码如下：
    private static final String[] READ_NOTE_PROJECTION = new String[] {
            NotePad.Notes._ID,               // Projection position 0, the note's id
            NotePad.Notes.COLUMN_NAME_NOTE,  // Projection position 1, the note's content
            NotePad.Notes.COLUMN_NAME_TITLE, // Projection position 2, the note's title
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, //Projection position 3,the note's modify date
    };
* 在OnCreate方法中，将时间装配进适配器，并显示到对应位置
    private static final String[] PROJECTION = new String[]{
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE,// 1
            NotePad.Notes.COLUMN_NAME_NOTE,//2 增加文本内容
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//3 增加修改时间
    };
    String[] dataColumns = {NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE};
    int[] viewIDs = {android.R.id.text1, R.id.text3};
* 上述时，显示的以毫秒为单位的一长串时间戳，需要对显示的时间进行数据处理。我采用的方法是，修改数据库中创建时间和修改时间的数据类型，使其直接存储为我们需要的时间显示格式。
  * 在NotePadProvider中先修改数据表中的创建时间和修改时间的数据类型
      @Override
       public void onCreate(SQLiteDatabase db) {
           db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                   + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                   + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " TEXT"
                   + ");");
       }
  * 在insert方法中修改初始化时的时间显示格式
      Long time_now = Long.valueOf(System.currentTimeMillis());

        SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String now = sdf.format(new Date(Long.parseLong(String.valueOf(time_now))));

        // If the values map doesn't contain the creation date, sets the value to the current time.
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, now);
        }

        // If the values map doesn't contain the modification date, sets the value to the current
        // time.
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, now);
        }
  * 在NoteEditor中updateNote方法中修改笔记更新时，修改时间的显示格式
      Long time_now = Long.valueOf(System.currentTimeMillis());

        SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String now = sdf.format(new Date(Long.parseLong(String.valueOf(time_now))));

        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, now);
      
      
