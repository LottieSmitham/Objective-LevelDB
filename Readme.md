## Introduction

A Objective-C database library built over [Google's LevelDB](http://code.google.com/p/leveldb), a fast embedded key-value store written by Google.

## Installation

By far, the easiest way to integrate this library in your project is by using [CocoaPods][1] if you're not already.

### Instructions for iOS Projects

1. Have [Cocoapods][1] installed
2. In your Podfile, add the line 

        pod 'Objective-LevelDB'

3. Run `pod install`
3. Make something awesome.

### Instructions for OS X projects

1. Have [Cocoapods][1] installed
2. In your Podfile, add the line 

        pod 'Objective-LevelDB'

3. Run `pod install`
4. From your project's root directory, go in `Pods/Objective-LevelDB/leveldb-library`
5. Run `make clean && make CC=clang CXX=clang++`
5. Make something awesome.

## How to use

### Creating/Opening a database file on disk

```objective-c
LevelDB *ldb = [LevelDB databaseInLibraryWithName:@"test.ldb"];
```

##### Setup Encoder/Decoder blocks

```objective-c
ldb.encoder = ^ NSData * (LeveldBKey *key, id object) {
  // return some data, given an object
}
ldb.decoder = ^ id (LeveldBKey *key, NSData * data) {
  // return an object, given some data
}
```

#####  NSMutableDictionary-like API

```objective-c
[ldb setObject:@"laval" forKey:@"string_test"];
NSLog(@"String Value: %@", [ldb objectForKey:@"string_test"]);

[ldb setObject:@{@"key1" : @"val1", @"key2" : @"val2"} forKey:@"dict_test"];
NSLog(@"Dictionary Value: %@", [ldb objectForKey:@"dict_test"]);
```
All available methods can be found in its [header file](Classes/LevelDB.h) (documented).

##### Enumeration

```objective-c
[self enumerateKeysAndObjectsUsingBlock:^(LevelDBKey *key, id value, BOOL *stop) {
    // This step is necessary since the key could be a string or raw data (use NSDataFromLevelDBKey in that case)
    NSString *keyString = NSStringFromLevelDBKey(key); // Assumes UTF-8 encoding
    // Do something clever
}];

// Enumerate with options
[self enumerateKeysAndObjectsBackward:TRUE
                               lazily:TRUE       // Block below will have a block(void) instead of id argument for value
                        startingAtKey:someKey    // Start iteration there (NSString or NSData)
                  filteredByPredicate:predicate  // Only iterate over values matching NSPredicate
                            andPrefix:prefix     // Only iterate over keys prefixed with something 
                           usingBlock:^(LevelDBKey *key, ^(^valueGetter)(void), BOOL *stop) {
                             
    NSString *keyString = NSStringFromLevelDBKey(key);
    id value = valueGetter();
}]
```
More iteration methods are available, just have a look at the [header section](Classes/LevelDB.h)

##### Snapshots, NSDictionary-like API (immutable)

A snapshot is a readonly interface to the database, permanently reflecting the state of 
the database when it was created, even if the database changes afterwards.

```objective-c
LDBSnapshot *snap = [ldb newSnapshot]; // You get ownership of this variable, so in non-ARC projects,
                                       // you'll need to release/autorelease it eventually
[ldb removeObjectForKey:@"string_test"];

// The result of these calls will reflect the state of ldb when the snapshot was taken
NSLog(@"String Value: %@", [snap objectForKey:@"string_test"]);
NSLog(@"Dictionary Value: %@", [ldb objectForKey:@"dict_test"]);

// get rid of it (non-ARC)
[snap release];
// ARC
snap = nil;
```

All available methods can be found in its [header file](Classes/Snapshot.h)

##### Write batches, atomic sets of updates

Write batches are a mutable proxy to a `LevelDB` database, accumulating updates
without applying them, until you do using `-[LDBWritebatch apply]`

```objective-c
LDBWritebatch *wb = [ldb newWritebatch];
[wb setObject:@{ @"foo" : @"bar" } forKey: @"another_test"];
[wb removeObjectForKey:@"dict_test"];

// Those changes aren't yet applied to ldb
// To apply them in batch, 
[wb apply];
```

All available methods can be found in its [header file](Classes/WriteBatch.h)

##### LevelDB options

```objective-c
// The following values are the default
LevelDBOptions options = [LevelDB makeOptions];
options.createIfMissing = true;
options.errorIfExists   = false;
options.paranoidCheck   = false;
options.compression     = true;
options.filterPolicy    = 0;      // Size in bits per key, allocated for a bloom filter, used in testing presence of key
options.cacheSize       = 0;      // Size in bytes, allocated for a LRU cache used for speeding up lookups

// Then, you can provide it when initializing a db instance.
LevelDB *ldb = [LevelDB databaseInLibraryWithName:@"test.ldb" andOptions:options];
```

##### Per-request options

```objective-c
db.safe = true; // Make sure to data was actually written to disk before returning from write operations.
[ldb setObject:@"laval" forKey:@"string_test"];
[ldb setObject:[NSDictionary dictionaryWithObjectsAndKeys:@"val1", @"key1", @"val2", @"key2", nil] forKey:@"dict_test"];
db.safe = false; // Switch back to default

db.useCache = false; // Do not use DB cache when reading data (default to true);
```

### License

Distributed under the [MIT license](LICENSE)

[1]: http://cocoapods.org