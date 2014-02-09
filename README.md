# MDMCoreData

A collection of lightweight Core Data classes for iOS and OS X.

[![Version](https://cocoapod-badges.herokuapp.com/v/MDMCoreData/badge.png)](http://cocoadocs.org/docsets/MDMCoreData)
[![Platform](https://cocoapod-badges.herokuapp.com/p/MDMCoreData/badge.png)](http://cocoadocs.org/docsets/MDMCoreData)

MDMCoreData is a growing collection of lightweight classes that make working with Core Data easier. It does not try to hide Core Data but instead tries to enforce best practices and reduce boiler plate code. All classes are documented and unit tested. 

* __MDMPersistenceController (iOS, OS X)__ - A lightweight class that sets up an efficient Core Data stack with support for creating multiple child managed object contexts. A private managed object context is used for asynchronous saving. A SQLite store is used for data persistence.
 
* __MDMFetchedResultsTableDataSource (iOS)__ -  A class mostly full of boiler plate that implements the fetched results controller delegate and a table data source and is used by a table view to access Core Data models.

* ...

|   | iOS | OS X | Documented | Tested  |
|--:|:-:|:-:|:-:|:-:|
| __MDMPersistenceController__         | ✓ | ✓ | ✓ | ✓ |
| __MDMFetchedResultsTableDataSource__ | ✓ |   | ✓ |   |
| ... |   |   |

![MDMCoreData Screenshot](https://github.com/mmorey/MDMCoreData/raw/master/screenshot_animated.gif)

## Usage

To run the example project clone the repo and open `MDMCoreData.xcworkspace`.

### MDMPersistenceController

To create a new `MDMPersistenceController` call `initWithStoreURL:modelURL:` with the URLs of the SQLite file and data model.

    NSURL *storeURL = [[self applicationDocumentsDirectory] URLByAppendingPathComponent:@"MDMCoreData.sqlite"];
    NSURL *modelURL = [[NSBundle mainBundle] URLForResource:@"MDMCoreData" withExtension:@"momd"];
    persistenceController = [[MDMPersistenceController alloc] initWithStoreURL:storeURL 
                                                                      modelURL:modelURL];
    
Easily access the main queue managed object context via the public `managedObjectContext` property:

    self.persistenceController.managedObjectContext

To save changes, call `saveContextAndWait:completion:` with an optional completion block:

    [self.persistenceController saveContextAndWait:NO completion:^(NSError *error) {
        
        if (error == nil) {
            NSLog(@"Successfully saved all the things!");
        }
    }];

New child context can be created with the main queue or a private queue for background work:

    NSManagedObjectContext *privateContextForScratchPadWork = [self.persistenceController newChildManagedObjectContext];
    
    NSManagedObjectContext *privateContextForDoingBackgroundWork = [self.persistenceController newPrivateChildManagedObjectContext];

### MDMFetchedResultsTableDataSource

To create a new `MDMFetchedResultsTableDataSource` call `initWithTableView:fetchedResultsController:` with a table view and fetched results controller. You also need to set the `delegate` and `reuseIdentifier`.

    self.tableDataSource = [[MDMFetchedResultsTableDataSource alloc] initWithTableView:self.tableView
                                                              fetchedResultsController:[self fetchedResultsController]];
    self.tableDataSource.delegate = self;
    self.tableDataSource.reuseIdentifier = @"Cell";
    self.tableView.dataSource = self.tableDataSource;

For cell configuration and object deletion `MDMFetchedResultsTableDataSource` requires all `MDMFetchedResultsTableDataSourceDelegate` methods be implemented:

	- (void)dataSource:(MDMFetchedResultsTableDataSource *)dataSource
         configureCell:(id)cell
            withObject:(id)object {
	    
	    UITableViewCell *tableCell = (UITableViewCell *)cell;
	    tableCell.textLabel.text = [[object valueForKey:@"timeStamp"] description];
	}
	
	- (void)dataSource:(MDMFetchedResultsTableDataSource *)dataSource 
	      deleteObject:(id)object 
	       atIndexPath:(NSIndexPath *)indexPath {
	    
	    [self.persistenceController.managedObjectContext deleteObject:object];
	}

During large data imports you can easily pause `MDMFetchedResultsTableDataSource` for improved performance:

    self.tableDataSource.paused = YES;

## Installation

MDMCoreData is available through [CocoaPods](http://cocoapods.org), to install it simply add the following line to your Podfile:

    pod "MDMCoreData"

If you don't need everything, you can install only what you need:

    pod "MDMCoreData/MDMPersistenceController"
    pod "MDMCoreData/MDMFetchedResultsTableDataSource"
    
To install manually just copy everything in the Classes directory into your Xcode project.

_**Important:**_ If your project doesn't use ARC you must add the `-fobjc-arc` compiler flag to all MDMCoreData implementation files in Target Settings > Build Phases > Compile Sources.

## Author

Created by [Matthew Morey](http://matthewmorey.com) and other [contributors](https://github.com/mmorey/MDMCoreData/graphs/contributors).

## License

MDMCoreData is available under the MIT license. See the [LICENSE](https://github.com/mmorey/MDMCoreData/LICENSE) file for more information. If you're using MDMCoreData in your project, attribution would be nice.

## Attribution

MDMCoreData is based on and inspired by the work of many:

* [objc.io Issue #4 Core Data](http://www.objc.io/issue-4/)
* [Marcus Zarra](https://twitter.com/mzarra)
* [High Performance Core Data](http://highperformancecoredata.com/)