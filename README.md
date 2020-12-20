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
   
# UI美化
--------
* 显示的内容中，再添加笔记的文本内容，并且多加一个星级条
  * 修改notelist_item.xml，修改后的代码为：
  
        <LinearLayout
              android:id="@+id/linear2"
              android:layout_width="0dp"
              android:layout_weight="2"
              android:layout_height="wrap_content"
              android:orientation="vertical">
              <TextView
                  android:id="@android:id/text1"
                  android:layout_width="match_parent"
                  android:layout_height="?android:attr/listPreferredItemHeight"
                  android:textAppearance="?android:attr/textAppearanceLarge"
                  android:textSize="30dp"
                  android:gravity="center_vertical"
                  android:paddingLeft="20dp"
                  android:singleLine="true"
              />

              <TextView
                  android:id="@android:id/text2"
                  android:layout_width="match_parent"
                  android:layout_height="?android:attr/listPreferredItemHeight"
                  android:textSize="20dp"
                  android:gravity="center_vertical"
                  android:paddingLeft="10dp"
                  android:singleLine="true"
                  />
          </LinearLayout>

          <LinearLayout
              android:layout_width="0dp"
              android:layout_weight="1"
              android:layout_height="wrap_content"
              android:orientation="vertical">
              <TextView
                  android:id="@+id/text3"
                  android:layout_width="wrap_content"
                  android:layout_height="wrap_content"
                  android:textColor="#00BFFF"/>
              <LinearLayout
                  android:layout_width="wrap_content"
                  android:layout_height="wrap_content"
                  android:orientation="vertical">
                  <RatingBar
                      android:id="@+id/rating"
                      android:layout_width="wrap_content"
                      android:layout_height="wrap_content"
                      android:numStars="3"
                      android:isIndicator="false"
                      android:rating="2"
                      android:stepSize="0.5"/>
              </LinearLayout>


          </LinearLayout>
   * 像显示时间戳一样，显示笔记的内容，将上述有关显示的地方，都修改代码为：
   
        String[] dataColumns = {NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_NOTE, NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE};

        int[] viewIDs = {android.R.id.text1, android.R.id.text2, R.id.text3};
* 将各条笔记分割开来，在Notelist中的onCreate方法中添加下列代码：

        ListView listView = getListView();
        listView.setCacheColorHint(0x00000000);
        listView.setDivider(new ColorDrawable(Color.BLACK));
        listView.setDividerHeight(20);
* 在编辑界面中，使得编辑的字体在编辑时能高亮显示，失去焦点后失去高亮
   * 在drawable中创建高亮显示的资源文件edit_image.xml，代码如下：
   
         <?xml version="1.0" encoding="utf-8"?>
         <selector xmlns:android="http://schemas.android.com/apk/res/android">
             <!-- 指定获得焦点时的颜色-->
             <item android:state_focused="true"
                 android:color="#f44"/>
             <!-- 指定失去焦点时的颜色-->
             <item android:state_focused="false"
                 android:color="#ccf"/>
         </selector>
   * 在note_editor和title_editor中，添加下面代码：
   
         android:textColor="@drawable/edit_image"
* 使得编辑标题后点击的确定按钮能聚焦
   * 在drawable中创建按钮的样式资源文件button_style.xml，代码如下：
   
         <?xml version="1.0" encoding="utf-8"?>
         <selector
             xmlns:android="http://schemas.android.com/apk/res/android">
             <item android:state_pressed="true" >
                 <shape>
                     <!-- 渐变 -->
                     <gradient
                         android:startColor="#ff8c00"
                         android:endColor="#FFFFFF"
                         android:type="radial"
                         android:gradientRadius="50" />
                     <!-- 描边 -->
                     <stroke
                         android:width="2dp"
                         android:color="#dcdcdc"
                         android:dashWidth="5dp"
                         android:dashGap="3dp" />
                     <!-- 圆角 -->
                     <corners
                         android:radius="2dp" />
                     <padding
                         android:left="10dp"
                         android:top="10dp"
                         android:right="10dp"
                         android:bottom="10dp" />
                 </shape>
             </item>

             <item android:state_focused="true" >
                 <shape>
                     <gradient
                         android:startColor="#ffc2b7"
                         android:endColor="#ffc2b7"
                         android:angle="270" />
                     <stroke
                         android:width="2dp"
                         android:color="#dcdcdc" />
                     <corners
                         android:radius="2dp" />
                     <padding
                         android:left="10dp"
                         android:top="10dp"
                         android:right="10dp"
                         android:bottom="10dp" />
                 </shape>
             </item>

             <item>
                 <shape>
                     <solid android:color="#ff9d77"/>
                     <stroke
                         android:width="2dp"
                         android:color="#fad3cf" />
                     <corners
                         android:topRightRadius="5dp"
                         android:bottomLeftRadius="5dp"
                         android:topLeftRadius="0dp"
                         android:bottomRightRadius="0dp"
                         />
                     <padding
                         android:left="10dp"
                         android:top="10dp"
                         android:right="10dp"
                         android:bottom="10dp" />
                 </shape>
             </item>
         </selector>
  * UI美化后的效果如图：
  ![UI整体图](https://github.com/Saberalter123/NotePad/blob/master/image/img11.png)
   * 编辑界面，编辑文本内容时的高亮和失去焦点时的对比：
    ![编辑内容无焦点](https://github.com/Saberalter123/NotePad/blob/master/image/img06.png)
    ![编辑内容有焦点](https://github.com/Saberalter123/NotePad/blob/master/image/img07.png) 
   * 编辑标题，有焦点和无无焦点的对比，按钮的效果显示：
    ![编辑标题无焦点](https://github.com/Saberalter123/NotePad/blob/master/image/img08.png)
    ![编辑标题无焦点](https://github.com/Saberalter123/NotePad/blob/master/image/img09.png) 
      * 点击按钮时，按钮颜色的变化效果：
      ![按钮的变化](https://github.com/Saberalter123/NotePad/blob/master/image/img10.jpg)
      
# 更改背景
----------
* 创建我们需要的背景资源文件background.xml,background_red.xml,background_green.xml,background_grey.xml,background_orange.xml,给背景设置圆角，增加描边，并且背景颜色为渐变色，它们的代码很相似，我以background_green.xml为例，代码如下：

       <?xml version="1.0" encoding="utf-8"?>
       <shape xmlns:android="http://schemas.android.com/apk/res/android"
           android:shape="rectangle"  android:scrollbarAlwaysDrawHorizontalTrack="false">
           <!--实心背景颜色-->
           <solid android:color="#AFEEEE" />
           <!--分别设置左上，右上，左下，右下-->
           <!-- 渐变 -->
           <gradient

               android:startColor="#AFEEEE"

               android:endColor="#54FF9F"

               android:angle="0"/>
           <!-- 描边 -->
           <stroke
               android:width="3dp"
               android:color="#dcdcdc"
               android:dashWidth="5dp"
               android:dashGap="3dp"/>
           <!--圆角-->
           <corners
               android:bottomLeftRadius="20dp"
               android:bottomRightRadius="20dp"
               android:topLeftRadius="30dp"
               android:topRightRadius="20dp" />
       </shape>
* 修改菜单list_option_menu.xml，添加一组选项，修改后的代码如下：

       <?xml version="1.0" encoding="utf-8"?>
       <menu xmlns:android="http://schemas.android.com/apk/res/android">
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
           <item android:title="background"
                 android:icon="@drawable/ic_menu_compose">
                 <menu>
                     <group>
                         <item
                             android:id="@+id/red_fond"
                             android:title="紫红色"/>
                         <item
                         android:id="@+id/greed_fond"
                         android:title="青绿色"/>
                         <item
                             android:id="@+id/grey_fond"
                             android:title="仿古色"/>
                         <item
                             android:id="@+id/orange_fond"
                             android:title="褐橘色"/>
                         <item
                             android:id="@+id/default_fond"
                             android:title="默认"/>
                     </group>
                 </menu>
           </item>

       </menu>
* 修改Notelist中的onOptionsItemSelected方法，添加菜单的响应事件，修改后的代码如下：

      @Override
          public boolean onOptionsItemSelected(MenuItem item) {
              if(item.isCheckable()){
                  item.setChecked(true);
              }
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
                  case R.id.red_fond:
                      ListView listView = getListView();
                      listView.setBackgroundResource(R.drawable.background_red);
                      return true;
                  case R.id.greed_fond:
                      ListView listView2 = getListView();
                      listView2.setBackgroundResource(R.drawable.background_green);
                      return true;
                  case R.id.grey_fond:
                      ListView listView3 = getListView();
                      listView3.setBackgroundResource(R.drawable.background_grey);
                      return true;
                  case R.id.orange_fond:
                      ListView listView4 = getListView();
                      listView4.setBackgroundResource(R.drawable.background_orange);
                      return true;
                  case R.id.default_fond:
                      ListView listView5 = getListView();
                      listView5.setBackgroundResource(R.drawable.background);
                      return true;
                  default:
                      return super.onOptionsItemSelected(item);
              }
          }
     
* 效果图如下：
   * 调出菜单选项：
   ![调出菜单](https://github.com/Saberalter123/NotePad/blob/master/image/img12.png) 
   * 改变颜色背景
   ![紫红色](https://github.com/Saberalter123/NotePad/blob/master/image/img13.png)
   ![青绿色](https://github.com/Saberalter123/NotePad/blob/master/image/img14.png)
   ![仿古色](https://github.com/Saberalter123/NotePad/blob/master/image/img15.png)
   ![褐橘色](https://github.com/Saberalter123/NotePad/blob/master/image/img16.png)
