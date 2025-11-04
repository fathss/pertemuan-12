## Bagian 1: Analisis Awal & Membaca diagram
1. Analisis Kode: Perhatikan kode PasswordReminder yang terikat pada MySQLConnection.
  - Apa yang akan terjadi pada kelas PasswordReminder jika method connect di MySQLConnection diubah parameternya menjadi connect(String user, String pass)? Di baris kode mana masalah akan muncul? <br>
 **Jawaban:**<br>
 Akan terjadi error parameter tidak lengkap karena pada kelas PasswordReminder hanya mengisi pada satu paramter saja. Error akan muncul pada baris kode ini:<br>
```java
public void sendReminders() {
  // Logika bisnis terikat erat dengan detail koneksi
  dbConnection.connect("jdbc:mysql://localhost/prod_db");
  ...
}
 ```
  - Apa yang akan terjadi pada kelas PasswordReminder jika method executeQuery di MySQLConnection diubah agar mengembalikan String[] (sebuah array) dan bukan List<String>? Perubahan apa yang harus Anda lakukan di PasswordReminder untuk menyesuaikannya?<br>
**Jawaban:**<br>
Akan terjadi error karena tipe data `userToRemind` adalah sebuah array list. Solusinya adalah merubah tipe data `userToRemind` menjadi sebuah array of string.
```java
if (dbConnection.isConnected()) {
  String[] usersToRemind = dbConnection.executeQuery("SELECT email FROM users WHERE reminder_due = true");
  ...
}
```

## Bagian 2: Misi Refactoring
#### Langkah 1:
`DBConnectionInterface.java`
```java
import java.util.List;

public interface DBConnectionInterface {
    void connect(String connectionInfo);
    void disconnect();
    List<String> executeQuery(String query);
    boolean isConnected();
}
```

#### Langkah 2:
`MySQLConnection.java`
```java
import java.util.List;
import java.util.Arrays;

public class MySQLConnection implements DBConnectionInterface {
    private boolean isConnected = false;

    @Override
    public void connect(String connectionString) {
        System.out.println("MySQL: Connecting using '" + connectionString + "'");
        this.isConnected = true;
        System.out.println("MySQL: Connection established.");
    }

    @Override
    public void disconnect() {
        System.out.println("MySQL: Connection closed.");
        this.isConnected = false;
    }

    @Override
    public List<String> executeQuery(String query) {
        if (!isConnected()) {
            throw new IllegalStateException("Not connected to MySQL database!");
        }
        System.out.println("MySQL: Executing query '" + query + "'");
        return Arrays.asList("user1@example.com", "user2@example.com");
    }

    @Override
    public boolean isConnected() {
        return this.isConnected;
    }
}
```

#### Langkah 3:
`PasswordReminder.java`
```java
import java.util.List;
import java.util.Arrays;

public class PasswordReminder {
    private final DBConnectionInterface dbConnection;

    // Suntikkan dependensi melalui constructor
    public PasswordReminder(DBConnectionInterface dbConnection) {
        this.dbConnection = dbConnection;
    }

    public void sendReminders() {
        // Logika bisnis tidak lagi tahu implementasi, hanya tahu kontrak!
        // Connection string sekarang menjadi tanggung jawab pemanggil.
        dbConnection.connect("jdbc:mysql://localhost/prod_db");
        
        if (dbConnection.isConnected()) {
            List<String> usersToRemind = dbConnection.executeQuery("SELECT email FROM users WHERE reminder_due = true");
            System.out.println("Found " + usersToRemind.size() + " users to remind.");

            for(String userEmail : usersToRemind) {
                System.out.println("Sending reminder to: " + userEmail);
            }
        }
        
        dbConnection.disconnect();
    }
}
```

#### Langkah 4:
`PostgreSQLConnection.java`
```java
import java.util.List;
import java.util.Arrays;

public class PostgreSQLConnection implements DBConnectionInterface {
    private boolean isConnected = false;
    
    @Override
    public void connect(String connectionInfo) {
        System.out.println("PostgreSQL: Connecting with '" + connectionInfo + "'");
        this.isConnected = true;
        System.out.println("PostgreSQL: Connection ready.");
    }

    @Override
    public void disconnect() {
        System.out.println("PostgreSQL: Disconnected.");
        this.isConnected = false;
    }

    @Override
    public List<String> executeQuery(String query) {
        if (!isConnected()) {
            throw new IllegalStateException("Not connected to PostgreSQL!");
        }
        System.out.println("PostgreSQL: Running query '" + query + "'");
        return Arrays.asList("pg_user_A@example.com", "pg_user_B@example.com");
    }

    @Override
    public boolean isConnected() {
        return this.isConnected;
    }
}
```

### Ekplorasi Tambahan
#### Bagian A: Implementasi & Penggunaan Koneksi "Simulasi"
1.  **Buat Koneksi "Simulasi":** Selain `MySQLConnection` dan `PostgreSQLConnection`, buatlah satu lagi kelas yang mengimplementasikan `DBConnectionInterface` bernama `SimulatedConnection`. Method `connect()` di dalam kelas ini tidak perlu terhubung ke database sungguhan, cukup mencetak pesan seperti `"SimulatedConnection: Simulasi koneksi untuk development."` dan method `executeQuery()` bisa mengembalikan `Arrays.asList("simulated_user1@example.com")`. Pastikan semua method `DBConnectionInterface` diimplementasikan.<br>
**Jawaban:**<br>
`SimulatedConnection.java`
```java
import java.util.List;
import java.util.Arrays;

public class SimulatedConnection implements DBConnectionInterface {
	private boolean isConnected = false;
    
    	@Override
    	public void connect(String connectionInfo) {
        	System.out.println("SimulatedConnection: Simulasi koneksi untuk development.");
        	this.isConnected = true; 
    		  System.out.println("SimulatedConnection: Connected.");   	
    	}
    
        	@Override
        	public void disconnect() {
            	System.out.println("SimulatedConnection: Disconnected.");  
    	        this.isConnected = false;
    	}
    
    	@Override
    	public List<String> executeQuery(String query) {
    	        if (!isConnected()) {
            	    throw new IllegalStateException("Not connected!");
    	        }
            	System.out.println("SimulatedConnection: Running query '" + query + "'");
    	        return Arrays.asList("simulated_user1@example.com");
    	}
    
    	@Override
    	public boolean isConnected() {
    	        return this.isConnected;
    	}
}
```
2.  **Gunakan Koneksi "Simulasi":** Coba jalankan `PasswordReminder` dengan menyuntikkan objek `SimulatedConnection` Anda di method `main`.<br>
**Jawaban:**<br>
`Main.java`
```java
public class Main {
    public static void main(String[] args) {
        ...
        System.out.println("--- SCENARIO 3: SIMULATING CONNECTION ---");
        // Membuat objek
        DBConnectionInterface simulation = new SimulatedConnection();
        // Menyuntikkan ke PasswordReminder
        PasswordReminder reminderSimulation = new PasswordReminder(simulation);
        reminderSimulation.sendReminders();
    }
}
```
3.  **Jelaskan Kegunaannya:** Menurut Anda, dalam situasi apa memiliki kelas "koneksi simulasi" seperti `SimulatedConnection` ini bisa sangat berguna selama proses pengembangan perangkat lunak?<br>
**Jawaban:**<br>
`SimulatedConnection` sangat berguna pada tahap pengujian. Dengan menggunakan koneksi simulasi ini, kita dapat meniru respons dari sebuah database tanpa perlu melakukan koneksi yang sebenarnya. Selain itu, koneksi simulasi ini memungkinkan kita menciptakan kondisi error secara sengaja dalam lingkungan yang terkontrol.

#### Bagian B: Refleksi Kritis terhadap Desain
Perhatikan method `sendReminders` di kelas `PasswordReminder` yang sudah di-refactor. Di dalamnya, kita masih menulis kode `dbConnection.connect("jdbc:mysql://localhost/prod_db");`.

1.  Meskipun `PasswordReminder` sekarang sudah tidak terikat pada kelas `MySQLConnection`, ia masih memiliki "pengetahuan" tentang `connection string` yang spesifik untuk MySQL. Menurut Anda, apakah ini masih bisa dianggap sebagai sebuah masalah atau "bau kode" (*code smell*) dalam desain kita? Jelaskan argumen Anda.<br>
**Jawaban:**<br>
Iya, ini masih dianggap sebagai masalah code smell. Ini karena, `PasswordReminder` seharusnya hanya bertanggung jawab pada bagian logika bisnis saja tanpa mengatur detail konfigurasi database.
2.  Tanpa perlu menulis kode, dapatkan Anda membayangkan sebuah cara atau strategi agar kelas `PasswordReminder` sama sekali tidak perlu tahu tentang detail `connectionString`? Jelaskan idenya secara konseptual.<br>
**Jawaban:**<br>
Solusi dari saya adalah membuat kelas `PasswordReminder` hanya berfokus pada logika bisnis saja. Sementara itu, konfigurasi koneksi database dilakukan di luar kelas PasswordReminder. Dari solusi ini, setiap kelas yang merepresentasikan jenis database (seperti MySQLConnection, PostgreSQLConnection, dan lainnya) dimodifikasi agar dapat menyimpan `connection string`-nya masing-masing. Dari cara ini nanti, method connect() tinggal mengambil nilai dari connection string yang sudah disimpan di dalam kelas tersebut. Nilai connection string akan ditetapkan ketika objek koneksi dibuat, sesuai dengan tipe database yang digunakan. Dengan pendekatan ini, PasswordReminder tidak lagi perlu mengetahui detail koneksi, dan dapat sepenuhnya fokus menjalankan fungsi logikanya

## Bagian 3: Pembuktian & Refleksi
#### Langkah 5: Uji Fleksibilitas Kode Anda Buat main method untuk membuktikan bahwa PasswordReminder bisa bekerja dengan kedua implementasi.<br>
`Main.java`
```java
public class Main {
    public static void main(String[] args) {
        System.out.println("--- SCENARIO 1: USING MYSQL ---");
        // Kita bisa membuat object MySQLConnection
        DBConnectionInterface mySQL = new MySQLConnection();
        // dan menyuntikkannya ke PasswordReminder
        PasswordReminder reminderMySQL = new PasswordReminder(mySQL);
        reminderMySQL.sendReminders();

        System.out.println("--- SCENARIO 2: SWITCHING TO POSTGRESQL ---");
        // Cukup ganti object yang kita suntikkan
        DBConnectionInterface postgreSQL = new PostgreSQLConnection();
        // PasswordReminder tidak perlu diubah sama sekali
        PasswordReminder reminderPostgreSQL = new PasswordReminder(postgreSQL);
        // Perhatikan bahwa connection string bisa berbeda untuk tiap database
        reminderPostgreSQL.sendReminders(); // Di dunia nyata, string ini akan berbeda

      	System.out.println("--- SCENARIO 3: SIMULATING CONNECTION ---");
      	// Membuat objek
        DBConnectionInterface simulation = new SimulatedConnection();
        // Menyuntikkan ke PasswordReminder
        PasswordReminder reminderSimulation = new PasswordReminder(simulation);
        reminderSimulation.sendReminders();
    }
}
```
**Output:**<br>
```shell
PS D:\pbo\teori\minggu11> javac *.java
PS D:\pbo\teori\minggu11> java Main
--- SCENARIO 1: USING MYSQL ---
MySQL: Connecting using 'jdbc:mysql://localhost/prod_db'
MySQL: Connection established.
MySQL: Executing query 'SELECT email FROM users WHERE reminder_due = true'
Found 2 users to remind.
Sending reminder to: user1@example.com
Sending reminder to: user2@example.com
MySQL: Connection closed.
--- SCENARIO 2: SWITCHING TO POSTGRESQL ---
PostgreSQL: Connecting with 'jdbc:mysql://localhost/prod_db'
PostgreSQL: Connection ready.
PostgreSQL: Running query 'SELECT email FROM users WHERE reminder_due = true'
Found 2 users to remind.
Sending reminder to: pg_user_A@example.com
Sending reminder to: pg_user_B@example.com
PostgreSQL: Disconnected.
--- SCENARIO 3: SIMULATING CONNECTION ---
SimulatedConnection: Simulasi koneksi untuk development.
SimulatedConnection: Connected.
SimulatedConnection: Running query 'SELECT email FROM users WHERE reminder_due = true'
Found 1 users to remind.
Sending reminder to: simulated_user1@example.com
SimulatedConnection: Disconnected.
```

#### Langkah 6: Jawab Pertanyaan Refleksi Dalam file jawaban.md Anda, jawablah pertanyaan-pertanyaan berikut:
Dalam file `jawaban.md` Anda, jawablah pertanyaan-pertanyaan berikut:

2.  Perhatikan *constructor* baru pada kelas `PasswordReminder`. Apa keuntungan utama dari memberikan objek dependensi (seperti `MySQLConnection`) melalui *constructor* dibandingkan membiarkan `PasswordReminder` membuatnya sendiri?<br>
**Jawaban:**<br>
Keuntungannya adalah `PasswordReminder` menjadi lebih fleksibel, mudah diuji, dan terhindar dari ketergantungan yang kaku.
3.  **Kaitkan dengan Class Diagram:** Perhatikan kembali **Class Diagram "Sesudah Refactoring"**. Jelaskan bagaimana hasil eksekusi kode di `main` method (Langkah 5) membuktikan bahwa `PasswordReminder` sekarang bergantung pada `DBConnectionInterface` dan bukan pada implementasi konkret (`MySQLConnection` atau `PostgreSQLConnection`). Fokus pada bagaimana panah dependensi pada diagram tercermin dalam fleksibilitas kode yang Anda amati.<br>
**Jawaban:**<br>
`PasswordReminder` sekarang bergantung pada `DBConnectionInterface` karena `PasswordReminder` tidak tahu implementasi apa yang diberikan. `PasswordReminder` hanya tahu bahwa objek yang diberikan tersebut memenuhi kontrak interface (punya connect(), disconnect(), isConnected(), dan executeQuery()). Dengan kata lain, `PasswordReminder` hanya menggunakan metode yang dijanjikan oleh interface `DBConnectionInterface`, tanpa mengetahui detail implementasinya.
