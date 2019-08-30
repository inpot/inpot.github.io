Flutter 学习

1. ### [dio](https://pub.dev/packages/dio) 网络请求

2. ### [fluttertoast](https://pub.dev/packages/fluttertoast) 显示Toast

   ```dart
   import 'package:fluttertoast/fluttertoast.dart';
   
   Fluttertoast.showToast(
           msg: "This is Center Short Toast",
           toastLength: Toast.LENGTH_SHORT,
           gravity: ToastGravity.CENTER,
           timeInSecForIos: 1,
           backgroundColor: Colors.red,
           textColor: Colors.white,
           fontSize: 16.0
       );
   
   ```

   

3. ### [url_launcher](https://pub.dev/packages/url_launcher) 打开web view

   ```dart
   import 'package:flutter/material.dart';
   import 'package:url_launcher/url_launcher.dart';
   
   void main() {
     runApp(Scaffold(
       body: Center(
         child: RaisedButton(
           onPressed: _launchURL,
           child: Text('Show Flutter homepage'),
         ),
       ),
     ));
   }
   
   _launchURL() async {
     const url = 'https://flutter.dev';
     if (await canLaunch(url)) {
       await launch(url);
     } else {
       throw 'Could not launch $url';
     }
   }
   
   
   ```

   

4. ### [path_provider](https://pub.dev/packages/path_provider) 文件路径

   ```dart
   Directory tempDir = await getTemporaryDirectory();
   String tempPath = tempDir.path;
   
   Directory appDocDir = await getApplicationDocumentsDirectory();
   String appDocPath = appDocDir.path;
   
   ```

   

5. ### [image_picker](https://pub.dev/packages/image_picker) 图片选择库

   ```
      var image = await ImagePicker.pickImage(source: ImageSource.camera);
   
   ```

   

6. ### [sqflite](https://pub.dev/packages/sqflite) sqlite

   ```dart
   import 'package:sqflite/sqflite.dart';
   var db = await openDatabase('my_db.db');
   await db.close();
   // Get a location using getDatabasesPath
   var databasesPath = await getDatabasesPath();
   String path = join(databasesPath, 'demo.db');
   
   // Delete the database
   await deleteDatabase(path);
   
   // open the database
   Database database = await openDatabase(path, version: 1,
       onCreate: (Database db, int version) async {
     // When creating the db, create the table
     await db.execute(
         'CREATE TABLE Test (id INTEGER PRIMARY KEY, name TEXT, value INTEGER, num REAL)');
   });
   
   // Insert some records in a transaction
   await database.transaction((txn) async {
     int id1 = await txn.rawInsert(
         'INSERT INTO Test(name, value, num) VALUES("some name", 1234, 456.789)');
     print('inserted1: $id1');
     int id2 = await txn.rawInsert(
         'INSERT INTO Test(name, value, num) VALUES(?, ?, ?)',
         ['another name', 12345678, 3.1416]);
     print('inserted2: $id2');
   });
   
   // Update some record
   int count = await database.rawUpdate(
       'UPDATE Test SET name = ?, VALUE = ? WHERE name = ?',
       ['updated name', '9876', 'some name']);
   print('updated: $count');
   
   // Get the records
   List<Map> list = await database.rawQuery('SELECT * FROM Test');
   List<Map> expectedList = [
     {'name': 'updated name', 'id': 1, 'value': 9876, 'num': 456.789},
     {'name': 'another name', 'id': 2, 'value': 12345678, 'num': 3.1416}
   ];
   print(list);
   print(expectedList);
   assert(const DeepCollectionEquality().equals(list, expectedList));
   
   // Count the records
   count = Sqflite
       .firstIntValue(await database.rawQuery('SELECT COUNT(*) FROM Test'));
   assert(count == 2);
   
   // Delete a record
   count = await database
       .rawDelete('DELETE FROM Test WHERE name = ?', ['another name']);
   assert(count == 1);
   
   // Close the database
   await database.close();
   
   ```

   

7. ### [cached_network_image](https://pub.dev/packages/cached_network_image) 图片缓存库

   ```D
   CachedNetworkImage(
           imageUrl: "http://via.placeholder.com/350x150",
           placeholder: (context, url) => new CircularProgressIndicator(),
           errorWidget: (context, url, error) => new Icon(Icons.error),
        ),
   
   Image(image: new CachedNetworkImageProvider(url))
   
   CachedNetworkImage(
     imageUrl: "http://via.placeholder.com/200x150",
     imageBuilder: (context, imageProvider) => Container(
       decoration: BoxDecoration(
         image: DecorationImage(
             image: imageProvider,
             fit: BoxFit.cover,
             colorFilter:
                 ColorFilter.mode(Colors.red, BlendMode.colorBurn)),
       ),
     ),
     placeholder: (context, url) => CircularProgressIndicator(),
     errorWidget: (context, url, error) => Icon(Icons.error),
   ),
   
   ```

   

8. ### [package_info](https://pub.dev/packages/package_info) pack信息

9. ```dart
   import 'package:package_info/package_info.dart';
   
   PackageInfo packageInfo = await PackageInfo.fromPlatform();
   
   String appName = packageInfo.appName;
   String packageName = packageInfo.packageName;
   String version = packageInfo.version;
   String buildNumber = packageInfo.buildNumber;
   ```

10. ### [webview_flutter](https://pub.dev/packages/webview_flutter)

11. ### [flutter_swiper](https://pub.dev/packages/flutter_swiper)

12. ### [photo_view](https://pub.dev/packages/photo_view)

13. ```dart
    import 'package:photo_view/photo_view.dart';
    @override
    Widget build(BuildContext context) {
      return Container(
        child: PhotoView(
          imageProvider: AssetImage("assets/large-image.jpg"),
        )
      );
    }
    ```

14. 

15. ### [flutter_easyrefresh](https://pub.dev/packages/flutter_easyrefresh)

16. ### [shared_preferences](https://pub.dev/packages/shared_preferences)

    ```dart
    import 'package:flutter/material.dart';
    import 'package:shared_preferences/shared_preferences.dart';
    
    void main() {
      runApp(MaterialApp(
        home: Scaffold(
          body: Center(
          child: RaisedButton(
            onPressed: _incrementCounter,
            child: Text('Increment Counter'),
            ),
          ),
        ),
      ));
    }
    
    _incrementCounter() async {
      SharedPreferences prefs = await SharedPreferences.getInstance();
      int counter = (prefs.getInt('counter') ?? 0) + 1;
      print('Pressed $counter times.');
      await prefs.setInt('counter', counter);
    }
    ```

    

17. ### [permission_handler](https://pub.dev/packages/permission_handler)

    ```dart
    import 'package:permission_handler/permission_handler.dart';
    ///requesting permission
    Map<PermissionGroup, PermissionStatus> permissions = await PermissionHandler().requestPermissions([PermissionGroup.contacts]);
    
    ///Checking permission
    PermissionStatus permission = await PermissionHandler().checkPermissionStatus(PermissionGroup.contacts);
    
    ///check service status
    ServiceStatus serviceStatus = await PermissionHandler().checkServiceStatus(PermissionGroup.location);
    ```

    ### 