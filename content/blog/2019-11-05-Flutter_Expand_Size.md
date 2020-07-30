Flutter Expand Size

1. Stack

   ```dart
   Stack( fit: StackFit.expand, children: <Widget>[
           Material(color: Colors.yellowAccent),
           Positioned(
             top: 0,
             left: 0,
             child: Icon(Icons.star, size: 50),
           ),
           Positioned(
             top: 340,
             left: 250,
             child: Icon(Icons.call, size: 50),
           ),
         ],
        ),
   ```

2. ConstrainedBox

   ```dart
   ConstrainedBox( 
     constraints: BoxConstraints.expand(),
     child: const Card(
       child: const Text('Hello World!'), 
       color: Colors.yellow,
     ), 
   ),
   ```

3. SizeBox.expand

   ```dart
   SizedBox.expand(
     child: Card(
       child: Text('Hello World!'),
       color: Colors.yellowAccent,
     ),
   ),
   ```

4. Row/Column

   ```dart
   Row(
     children: <Widget>[
       Expanded(
         child: Container(
           decoration: const BoxDecoration(color: Colors.red),
         ),
         flex: 3,
       ),
       Expanded(
         child: Container(
           decoration: const BoxDecoration(color: Colors.green),
         ),
         flex: 2,
       ),
       Expanded(
         child: Container(
           decoration: const BoxDecoration(color: Colors.blue),
         ),
         flex: 1,
       ),
     ],
   ),
   ```

   
