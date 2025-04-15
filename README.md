# Flutter Objectbox

1. Membuat Model Database

   Buat folder Model di dalam folder lib, kemudian buat model database untuk class User

   ---
       import 'package:objectbox/objectbox.dart';
  
        @Entity()
        // @Sync()
        class User {
          int id;
          String name;
          String email;
        
          User({
            this.id = 0,
            required this.name,
            required this.email,
          });
        }
   ---
   
   
3. Menambahkan Dependencies

   Dependencies dapat ditambahkan di file pubxpec.yaml, dependencies yang dibutuhkan antara lain sebagai berikut

   ---
         dependencies:
           flutter:
             sdk: flutter
           cupertino_icons: ^1.0.8
           objectbox: ^4.1.0
           objectbox_flutter_libs: any
           path_provider: ^2.0.11
           faker: ^2.2.0
         
         dev_dependencies:
           flutter_test:
             sdk: flutter
           flutter_lints: ^5.0.0
           build_runner: ^2.3.0
           objectbox_generator: ^4.1.0
   ---

   - Dependencies
     
   | Package               | Kegunaan                                               |
   |-----------------------|--------------------------------------------------------|
   | objectbox             | Database lokal yang akan digunakan                     |
   | objectbox_flutter_libs| Binding dan dukungan platform ObjectBox untuk Flutter agar bisa digunakan pada berbagai platform (Android / IOS)  |
   | path_provider         | digunakan untuk mengakses direktori pada perangkat jika diperlukan       |
   | faker                 | Generate data palsu (dummy data) untuk testing         |

   - Dev Dependencies
   
   | Package               | Kegunaan                                               |
   |-----------------------|--------------------------------------------------------|
   | build_runner          | Tool untuk generate code otomatis dan dapat digunakan untuk ObjectBox       |
   | objectbox_generator   | Plugin untuk build_runner yang digunakan untuk menghasilkan file database ObjectBox berdasarkan model database         |


   Setelah menambahkan dependencies pada pubxpec.yaml, jalankan command berikut pada terminal

   ---

         flutter pub get
         flutter pub run build_runner build

   ---

   
5. Aplikasi

   ---
         import 'package:faker/faker.dart';
         import 'package:flutter/material.dart';
         import 'package:flutter_objectbox/helper/object_box.dart';
         import 'package:flutter_objectbox/model/user.dart';
         
         late ObjectBox objectBox;
         
         Future main() async {
           WidgetsFlutterBinding.ensureInitialized();
           objectBox = await ObjectBox.init();
         
           runApp(const MyApp());
         }
         
         class MyApp extends StatelessWidget {
           const MyApp({Key? key}) : super(key: key);
         
           @override
           Widget build(BuildContext context) => MaterialApp(
                 title: 'ObjectBox',
                 debugShowCheckedModeBanner: false,
                 theme: ThemeData(
                   colorSchemeSeed: Colors.cyan,
                   textTheme: const TextTheme(
                     bodyMedium: TextStyle(fontSize: 20),
                     titleMedium: TextStyle(fontSize: 25, fontWeight: FontWeight.bold),
                   ),
                 ),
                 home: const Homepage(),
               );
         }
         
         class Homepage extends StatefulWidget {
           const Homepage({Key? key}) : super(key: key);
         
           @override
           State<Homepage> createState() => _HomepageState();
         }
         
         class _HomepageState extends State<Homepage> {
           late Stream<List<User>> streamUsers;
         
           @override
           void initState() {
             super.initState();
         
             streamUsers = objectBox.getUsers();
           }
         
           @override
           Widget build(BuildContext context) => Scaffold(
                 appBar: AppBar(
                   title: const Text('ObjectBox'),
                   centerTitle: true,
                 ),
                 body: StreamBuilder<List<User>>(
                   stream: streamUsers,
                   builder: (context, snapshot) {
                     if (!snapshot.hasData) {
                       return const Center(
                         child: CircularProgressIndicator(),
                       );
                     } else {
                       final users = snapshot.data!;
         
                       return ListView.builder(
                         itemCount: users.length,
                         itemBuilder: (context, index) {
                           final user = users[index];
         
                           return ListTile(
                             title: Text(user.name),
                             subtitle: Text(user.email),
                             trailing: IconButton(
                               icon: const Icon(Icons.delete),
                               onPressed: () => objectBox.deleteUser(user.id),
                             ),
                             onTap: () {
                               user.name = Faker().person.firstName();
                               user.email = Faker().internet.email();
         
                               objectBox.insertUser(user);
                             },
                           );
                         },
                       );
                     }
                   },
                 ),
                 floatingActionButton: FloatingActionButton(
                   child: const Icon(Icons.add),
                   onPressed: () {
                     final user = User(
                       name: Faker().person.firstName(),
                       email: Faker().internet.email(),
                     );
         
                     objectBox.insertUser(user);
                   },
                 ),
               );
         }
   ---
   
   ---

         import 'dart:io';
         
         import '../model/user.dart';
         import '../objectbox.g.dart';
         
         class ObjectBox {
           late final Store _store;
           late final Box<User> _userBox;
         
           ObjectBox._init(this._store) {
             _userBox = Box<User>(_store);
           }
         
           static Future<ObjectBox> init() async {
             final store = await openStore();
         
             // if (Sync.isAvailable()) {
             //   // jika menggunakan physical device bisa pake IP dari server
             //   // final ipSyncServer = '{ip dari server / komputernya}'
             //   final ipSyncServer = Platform.isAndroid ? '10.0.2.2' : '127.0.0.1';
             //   final syncClient = Sync.client(
             //     store,
             //     'ws://$ipSyncServer:9999',
             //     SyncCredentials.none(),
             //   );
             //   syncClient.connectionEvents.listen(print);
             //   syncClient.start();
             // }
         
             return ObjectBox._init(store);
           }
         
           User? getUser(int id) => _userBox.get(id);
         
           Stream<List<User>> getUsers() => _userBox
               .query()
               .watch(triggerImmediately: true)
               .map((query) => query.find());
         
           int insertUser(User user) => _userBox.put(user);
         
           bool deleteUser(int id) => _userBox.remove(id);
         }
      
   ---
