# 期中实验项目NotePad
 This is an AndroidStudio rebuild of google SDK sample NotePad
# NotePad主要目录结构
 ------------------
 ![项目结构](https://github.com/Saberalter123/NotePad/blob/master/image/img.png "项目结构图")
 
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
      
* 修改后，显示的时间效果如下：
![显示时间戳1](https://github.com/Saberalter123/NotePad/blob/master/image/img01.png)
![显示时间戳2](https://github.com/Saberalter123/NotePad/blob/master/image/img02.png)

# 笔记搜索功能
-------------
* 要添加笔记查询功能，就要在应用中增加一个搜索的入口。在menu文件夹下创建search_menu.xml文件来配置菜单栏的搜索框及样式，其中搜索小图标为自己在网上找到的小图标，搜索框代码如下：

       <?xml version="1.0" encoding="utf-8"?>
       <menu xmlns:android="http://schemas.android.com/apk/res/android"
           xmlns:app="http://schemas.android.com/apk/res-auto"
           >
           <item
               android:id="@+id/search"
               android:icon="@drawable/search"
               android:title="Search"
               android:actionViewClass="android.widget.SearchView"
               android:showAsAction="always"
               ></item>
       </menu>
* 在NoteList类中的onCreateOptionsMenu方法将第一步定义的菜单栏资源对象进行获取并实例化为SearchView对象，然后进行搜索框相关属性的设置，再对提交、输入值的响应事件进行监听与处理，并设置点击关闭按钮退出搜索，最终完成搜索框的模糊查询，退出后显示全部的笔记，实现代码如下：
  * 实例化搜索框，并通过SearchView对象进行搜索框提示词的设置
    
        getMenuInflater().inflate(R.menu.search_menu, menu);
        SearchView mysearchview = menu.findViewById(R.id.search).getActionView();
        mysearchview.setQueryHint("搜索");
  * 设置搜索框的监听事件
      
        mysearchview.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
              //当搜索框被提交时的响应事件
            @Override
            public boolean onQueryTextSubmit(String query) {
                search(query);

                return false;
            }

            //当搜索框中的文本内容改变时的响应事件
            @Override
            public boolean onQueryTextChange(String query) {

                return false;
            }

        });
  * 设置退出搜索时的响应事件
    
        mysearchview.setOnCloseListener(new SearchView.OnCloseListener() {
                @Override
                public boolean onClose() {
                    refresh();
                    return false;
                }
            });
  * 对提交的信息进行模糊查询，并返回查询的结果
 
        void search(String key) {
               Intent intent = getIntent();


            if (intent.getData() == null) {
                intent.setData(NotePad.Notes.CONTENT_URI);
            }

            getListView().setOnCreateContextMenuListener(this);

            //通过设置查询条件，达到模糊查询的实现
            String selection = NotePad.Notes.COLUMN_NAME_TITLE + " LIKE ?";
            String[] selectionArgs = {"%" + key + "%"};

            Cursor cursor = managedQuery(
                    getIntent().getData(),
                    PROJECTION,
                    selection,
                    selectionArgs,
                    NotePad.Notes.DEFAULT_SORT_ORDER
            );

            String[] dataColumns = {NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE};

            int[] viewIDs = {android.R.id.text1, android.R.id.text2};

            SimpleCursorAdapter adapter
                    = new SimpleCursorAdapter(
                    this,                             
                    R.layout.noteslist_item,          
                    cursor,                           
                    dataColumns,
                    viewIDs
            );
            setListAdapter(adapter);

        }
   * 退出后重新加载所有的笔记
   
         void refresh() {
              Intent intent = getIntent();


              if (intent.getData() == null) {
                  intent.setData(NotePad.Notes.CONTENT_URI);
              }

              getListView().setOnCreateContextMenuListener(this);

              String selection = null;
              String[] selectionArgs = null;

              Cursor cursor = managedQuery(
                      getIntent().getData(),
                      PROJECTION,
                      selection,
                      selectionArgs,
                      NotePad.Notes.DEFAULT_SORT_ORDER
              );

              String[] dataColumns = {NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE};

              int[] viewIDs = {android.R.id.text1, android.R.id.text2};

              SimpleCursorAdapter adapter
                      = new SimpleCursorAdapter(
                      this,                             
                      R.layout.noteslist_item,          
                      cursor,                           
                      dataColumns,
                      viewIDs
              );
              setListAdapter(adapter);


          }
   * 搜索效果如图：
   ![搜索1](https://github.com/Saberalter123/NotePad/blob/master/image/img03.png)
   ![搜索2](https://github.com/Saberalter123/NotePad/blob/master/image/img04.png)
   ![搜索3](https://github.com/Saberalter123/NotePad/blob/master/image/img05.png)
