# Flutter Objectbox

1. Membuat Model Database
   
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
4. Aplikasi
