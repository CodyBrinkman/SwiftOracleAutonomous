# SwiftOracleAutonomous
Driver for Swift to Oracle Autonomous Database

This builds on top of https://github.com/goloveychuk/SwiftOracle and wraps ocilib (https://github.com/vrogier/ocilib)

Shoutout to Vincent Rogier (vrogier) himself for assistance on this project.

_This example uses an Ubuntu 18.04 instance_

1. Download Oracle Client (this example uses v19.3)
```
oracle-instantclinet*-basic-*.rpm
oracle-instantclinet*-devel-*.rpm
oracle-instantclinet*-sqlplus-*.rpm
```

2. Install RPMs
```
$ sudo alien -i oracle-instantclient*-basic-*.rpm
$ sudo alien -i oracle-instantclinet*-devel-*.rpm
$ sudo alien -i oracle-instantclinet*-sqlplus-*.rpm
```

3. Install libaio1
```
$ sudo apt install libaio1
```

4. Set environment variables
```
$ export ORACLE_HOME=/usr/lib/oracle/19.3/client64
$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib/oracle/19.3/client64/lib:/usr/local/lib
```

5. Download and configure OCILIB library
```
$ git clone https://github.com/vrogier/ocilib.git
$ tar -zxf ocilib-4.5.2-gnu.tar.gz
$ cd ocilib-4.5.2
$ ./configure --with-oracle-headers-path=/usr/include/oracle/19.3/client64/ --with-oracle-lib-path=/usr/lib/oracle/19.3/client64/lib CFLAGS="-O2 -m64"
$ make
$ sudo make install
```

6. Configure wallet

unzip
```
$ unzip <wallet> -d <directory-to-make name>
```
set TNS_NAMES environment variable
```
$ export TNS_ADMIN=<path to wallet>
```
_To have TNS_ADMIN environment variable persist multiple sessions add the path to ~/.bashrc_

set wallet location to TNS_ADMIN
```
$ cd <wallet directory>
$ vi sqlnet.ora
```
set DIRECTORY to "$TNS_ADMIN". Your sqlnet.ora file should look like this:
```
WALLET_LOCATION = (SOURCE = (METHOD = file) (METHOD_DATA = (DIRECTORY="$TNS_ADMIN")))
SSL_SERVER_DN_MATCH=yes
```

7. Adding to Vapor project

Package.swift
```swift
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "VaporAutonomousConnection",
    products: [
        .library(name: "VaporAutonomousConnection", targets: ["App"]),
    ],
    dependencies: [
        // 💧 A server-side Swift web framework.
        .package(url: "https://github.com/vapor/vapor.git", from: "3.0.0"),

        // 🔵 Swift ORM (queries, models, relations, etc) built on SQLite 3.
        .package(url: "https://github.com/vapor/fluent-sqlite.git", from: "3.0.0"),

	// Vapor Oracle Autonomous Database driver
	.package(url: "https://github.com/CodyBrinkman/SwiftOracleAutonomous.git", from: "2.0.4")
    ],
    targets: [
        .target(name: "App", dependencies: ["FluentSQLite", "SwiftOracleAutonomous", "Vapor"]),
        .target(name: "Run", dependencies: ["App"]),
        .testTarget(name: "AppTests", dependencies: ["App"])
    ]
)
```

routes.swift
```swift
import Vapor
import SwiftOracleAutonomous

let b = Connection(connectString: "<database connection string>", user:"<username>", pwd: "<your password>")

final class ATPData {

	func getData() -> [[String:String]] {	
		try! b.open()
		b.autocommit = true
		let cursor = try! b.cursor()
		try! cursor.execute("select * from test")
		let items = cursor.map { row in row.dict.mapValues { "\($0 ?? "NULL")" }} //  output as [[String:String]]
		return items

	}
}

/// Register your application's routes here.
public func routes(_ router: Router) throws {
    // Basic "It works" example
    router.get { req in
        return "It works!"
    }
    
    // Basic "Hello, world!" example
    router.get("hello") { req in
        return "Hello, world!"
    }

    //data route
    router.get("test") { req -> [[String:String]] in
	let atp = ATPData()
	let atpdata = atp.getData()
	return atpdata
    }

    // Example of configuring a controller
    let todoController = TodoController()
    router.get("todos", use: todoController.index)
    router.post("todos", use: todoController.create)
    router.delete("todos", Todo.parameter, use: todoController.delete)
}
```
* connectString: Found in tnsnames.ora. My database name is SwiftATP so an example connectString is "swiftatp_high"
* user: Database username. Example is "ADMIN"
* pwd: Database password

To run Vapor 3 you need to link the library files
```
$ swift build -Xlinker -L/usr/local/lib && ./.build/x86_64-unknown-linux/debug/Run --hostname 0.0.0.0
```
