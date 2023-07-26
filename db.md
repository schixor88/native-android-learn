**Navigation** :

- [Home](index.md)

---

# Local Database with SQLite

### What is database?

- structured collection of data
- which is organized
- which is stored
- which is managed
  - in such a way that it is easy to
    - retrieve
    - manipulate
    - query
    - analyse
- a central repository where information lives which

  - can be accessed by
    - users
    - applications

- Core existence of database is to manage vast amount of data
- No organization can exist without a database

- In the world database can be:

  - local
    - in the physical device
    - in your own environment
    - in your laptop, in your pc
  - remote
    - in a cloud
    - in a remote server
    - in machine which is not in your local domain

- In mobile programming databases are
  - local database
  - inside the application memory
  - sqlite
  - structured query language - lite version

### Connectivity, Database, Table, Manipulate

How can we achieve database connectivity and data manipulation using SQLite database in our native android application?

> Lets learn by code example to create a sqlite database in your android application

Steps:

1. Make a class which `extends` the `SQLiteOpenHelper`
   ```java
   public class DBHandler extends SQLiteOpenHelper {
   ```
   - also notice that you'll have to override two methods
     - onCreate
     - onUpgrade
2. Define `String` variables needed for database to exist

   - database name
   - database version
   - table name (can be multiple)
   - column names (can be multiple)

   ```java
   private static final String DB_NAME = "asian";
   //any change to db should change/ increase the version
   private static final int DB_VERSION = 1;
   private static final String TABLE_NAME = "course";
   private static final String ID_COL = "id";
   private static final String NAME_COL = "name";
   private static final String DURATION_COL = "duration";
   ```

3. Create a constructor where super takes the database name and the database version which you've declared in step 2

   ```java
   public DBHandler(Context context) {
       super(context, DB_NAME, null, DB_VERSION);
   }

   ```

4. Create your table inside the method `onCreate()` which we had to `@override` from `SQLiteOpenHelper`

   ```java
   String query = "CREATE TABLE " + TABLE_NAME + " ("
               + ID_COL + " INTEGER PRIMARY KEY AUTOINCREMENT,"
               + NAME_COL + " TEXT,"
               + DURATION_COL + " TEXT)";
       sqLiteDatabase.execSQL(query);
   ```

5. Now we have a table where we can save course-name and course-duration

   - so we will make a function which can take
     - courseName
     - courseDuration
   - create an object to get instance of database
   - create an object to take value and put into database
   - insert that value inside the database
   - close the database

   ```java
    public void addNewCourse(String courseName, String courseDuration){
       //variable to write data in db
       SQLiteDatabase db = this.getWritableDatabase();
       //variable for values we will be putting in db
       ContentValues values = new ContentValues();
       //passing values as key-value pairs
       values.put(NAME_COL, courseName);
       values.put(DURATION_COL, courseDuration);
       //process the values to table
       db.insert(TABLE_NAME, null, values);
       // at last we are closing our
       // database after adding database.
       db.close();
   }

   ```

6. Now after this function we create another function which can retrieve the data from our course database called `getAllCourse()` - for database Cursor is a very important mechanism - it helps to `traverse` and `manipulate` the result set of query - it helps to `navigate` between the rows returned by the database query - cursor serves as a `pointer` to any incoming result set which can `interact` with the `result` set rows one by one - cursor can be opened on a specific query to get the rows `sequentially` or `selectively` - when not in use the opened cursor are closed.

   ```java
   public List<Map<String, String>> getAllCourses() {
   //making list of map of s,s
   List<Map<String, String>> list1 = new ArrayList<>();
   //variable to read data from db
   SQLiteDatabase db = this.getReadableDatabase();
   //query to read data
   String selectQuery = "SELECT \* FROM " + TABLE_NAME;
   //cursor
   Cursor cursor = db.rawQuery(selectQuery, null);
   //looping through cursor from all rows and adding the value to a list
   if (cursor.moveToFirst()) {
   do {
   Map<String, String> map = new HashMap<>();
   String courseId = cursor.getString(0);
   String courseName = cursor.getString(1);
   String courseDuration = cursor.getString(2);
   //
   map.put("id", courseId);
   map.put("name", courseName);
   map.put("duration", courseDuration);
   //
   list1.add(map);
   } while (cursor.moveToNext());
   }
   cursor.close();
   return list1;
   }

   ```

7. Completing the remaining `@override` method
   - the `onUpgrade` method is called when - the database needs to be upgraded to a new version - typically due to changes in the database schema.
   - primary use of the `onUpgrade` method is
     - to handle changes to the database structure
       - such as adding new tables
       - modifying existing tables
       - changing the data in the database.
   - crucial when you release a new version of your Android app that includes changes to the database schema
     - as you need to ensure that existing users can smoothly transition to the updated version without losing their data.
   - how it works
     - When you release an update to your app with changes to the database schema (e.g., adding new tables or altering existing ones),
       - you need to increment the database version number in your `SQLiteOpenHelper` subclass constructor.
       - This signals to Android that the database needs to be upgraded.
     - When a user installs the updated version of the app
       - Android checks if the database version specified in the code is higher than the version of the currently installed database on the device.
     - If the database version is higher
       - Android calls the `onUpgrade` method, passing:
         - the old database version
         - the new database version
         - and a database object.
     - Inside the `onUpgrade` method, you can implement the necessary changes to update the database schema or perform any data migration as needed.
       - This could involve
         - creating new tables,
         - altering existing ones
         - transferring data from the old schema to the new schema.
     - After the `onUpgrade` method completes, Android continues with the normal flow, allowing the updated app to use the updated database schema.
     ```java
     @Override
     public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
       //drops if table is existing
           sqLiteDatabase.execSQL("DROP TABLE IF EXISTS " + TABLE_NAME);
           onCreate(sqLiteDatabase);
       }
     ```
8. In your activity where you need to perform database logics like save or get data
   - create a database handler object as
     ```java
       dbHandler = new DBHandler(MainActivity.this);
     ```
   - to save a new course (usually inside a button action event)
     ```java
     dbHandler.addNewCourse(
     	courseName.getText().toString(),
     	courseDuration.getText().toString()
     );
     ```
   - to get all the values (also inside action)
     ```java
     List<Map<String, String>> courses = dbHandler.getAllCourses();
               for (Map<String, String> item :
                       courses) {
                   Log.d(TAG, "items : " + item.get("id") + "\n");
                   Log.d(TAG, "items : " + item.get("name") + "\n");
                   Log.d(TAG, "items : " + item.get("duration") + "\n\n");
               }
     ```
