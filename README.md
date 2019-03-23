# LiveCodingArchive
年次制作  
「RDBMSを用いたライブコーディングのアーカイヴシステム」のライブラリの一部
  
  
[ソース](https://github.com/KakuyaShiraishi/LiveCodingArchive/blob/master/MySqliteLib.h)  
      
    
      
  
  
## Open database

    #include <MySqliteLib.h>
    using namespace MySqliteLib;
    auto db = Sqlite('./test.db');

## Create table

    db.prepare(
        "CREATE TABLE IF NOT EXISTS people ("
        "    id INTEGER PRIMARY KEY AUTOINCREMENT,"
        "    name TEXT,"
        "    age INTEGER,"
        "    data BLOB"
        ");"
        ).execute();

## Drop table

    db_.prepare("DROP TABLE IF EXISTS people").execute();

## Insert records

    auto stmt = db_.prepare("INSERT INTO people (name, age, data) VALUES (?, ?, ?)");
    stmt.execute("kaku", 10, vector<char>({ 'A', 'B', 'C', 'D' }));
    stmt.execute("ryot", 20, vector<char>({ 'E', 'B', 'G', 'H' }));
    stmt.execute("tosh", 15, vector<char>({ 'I', 'J', 'K', 'L' }));
    stmt.execute("riki", 25, vector<char>({ 'M', 'N', 'O', 'P' }));

## Select a record (single colum)

    auto sql = "SELECT age FROM people WHERE name='kaku'";
    auto val = db_.prepare<Int>(sql).execute_value(); // 10

## Select records (multiple columns)

    auto sql = "SELECT age, name FROM people";

    auto rows = db_.prepare<Int, Text>(sql).execute(); 
    rows.size(); // 4

    auto row = rows[3];
    get<0>(row); // 25
    get<1>(row); // riki

## Bind #1

    auto sql = "SELECT name FROM people WHERE age > ?";
    auto stmt = db_.prepare<Text>(sql);

    {
        auto rows = stmt.execute(10);
        rows.size(); // 3
        rows[0];     // ryot
    }

    {
        auto rows = stmt.bind(10).execute();
        rows.size(); // 3
        rows[0];     // ryot
    }

## Bind #2

    auto sql = "SELECT id FROM people WHERE name=? AND age=?";
    auto val = db_.prepare<Int>(sql).execute_value("kaku", 10);
    val; // 1

## Cursor (multiple columns)

    auto sql = "SELECT name, age FROM people";
    auto stmt = db_.prepare<Text, Int>(sql);

    auto cursor = stmt.execute_cursor();
    auto it = cursor.begin();

    while (it != cursor.end()) {
        get<0>(*it); 
        get<1>(*it); 
        ++it;
    }

    for (const auto& x: stmt.execute_cursor()) {
        get<0>(x); 
        get<1>(x); 
    }

## Cursor (single column)

    auto sql = "SELECT name FROM people";
    auto stmt = db_.prepare<Text>(sql);

    auto itData = data_.begin();
    for (const auto& x: stmt.execute_cursor()) {
        x;
        ++itData;
    }

## Count

    auto sql = "SELECT COUNT(*) FROM people";
    auto val = db_.prepare<Int>(sql).execute_value();
    val; // 4

## Flat API

    auto val = db_.execute_value<Int>("SELECT COUNT(*) FROM people");
    auto rows = db_.execute<Text>("SELECT name FROM people WHERE age > ?", 10);
    auto rows2 = db_.execute<Int, Text>("SELECT age, name FROM people");
    db_.execute("DROP TABLE IF EXISTS test;");
