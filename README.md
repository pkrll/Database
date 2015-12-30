## Database [![Latest Stable Version](https://poser.pugx.org/saturn/taurus/v/stable)](https://packagist.org/packages/saturn/database) [![License](https://poser.pugx.org/saturn/database/license)](https://packagist.org/packages/saturn/database)
A PDO Wrapper Class.
### Installation
##### With Composer (recommended):
```bash
$ composer require saturn/database
```
#### Setup
Define and set the following constants:
```php
define("kHostname", "hostname"); // The hostname on which the database server resides.
define("kDatabase", "database"); // The name of the database.
define("kUsername", "username"); // The username.
define("kPassword", "password"); // The password.
```
### Usage
Using Database is easy. Just include it (either manually or using Composer's loader), and create an instance by supplying the above constants. See [below](#examples) for more examples.
```php
use saturn\database\Database;

$database = new Database($hostname, $database, $username, $password);

// Retrieve from the database
$SQLQuery = "SELECT SomeColumn FROM SomeTable";
$response = $database->read($SQLQuery);

// Insert into the database
$SQLQuery = "INSERT INTO SomeTable (SomeColumn) VALUES (:someValue)";
$SQLParam = array("someValue" => "Foobar");
$response = $database->write($SQLQuery, $SQLParam);
```
### API
Below outlines the public methods of the Database class.
```php
/*
 *  Create the PDO instance and set the defaults attributes on the database handle.
 *
 *  @param      string  Database hostname.
 *  @param      string  Database name.
 *  @param      string  Username.
 *  @param      string  Password.
 */
public function __construct($hostname, $database, $username, $password) {}

/**
 *  Run the supplied query. Only for fetching rows from
 *  the database.
 *
 *  @param      string  Optional. The SQL query to execute.
 *  @param      array   Optional. Additional parameters to supply to the query.
 *  @param      bool    If true, fetches all matching rows. Defaults to TRUE.
 *  @return     array
 **/
public function read($query, $params = NULL, $shouldFetchAll = true) {}

/**
 *  Run the supplied query. Only for adding rows to the the database.
 *
 *  @param      string  Optional. The SQL query to execute.
 *  @param      array   Optional. Additional parameters to supply to the query.
 *  @return     array
 **/
public function write($query = NULL, $params = NULL) {}

/**
 *  Return an array of error information about the last performed operation.
 *
 *  @param      bool    Value determines if the errorInfo should be performed on the 
 *                      database handle or the statement handle.
 *  @return     array
 */
public function error($connection = true) {}

/**
 *  Execute the prepared SQL statement.
 *
 *  @param      array   Optional. The input parameters.
 *  @return     mixed
 */
public function execute($params = NULL) {}

/**
 *  Fetch all the rows in the result set.
 *
 *  @param      int     Optional. Value controls how the row should be returned. The value 
 *                      must be one of the FETCH_* constants. Defaults to: FETCH_ASSOC.
 *  @return     mixed
 */
public function fetchAll($flags = PDO::FETCH_ASSOC) {}

/**
 *  Fetch the next row in the result set.
 *
 *  @param      int     Optional. Value controls how the row should be returned. The value 
 *                      must be one of the FETCH_* constants. Defaults to: FETCH_ASSOC.
 *  @return     mixed
 */
public function fetch($flags = PDO::FETCH_ASSOC) {}

/**
 * Prepares a statement for execution.
 *
 *  @param      string  The SQL string.
 *  @return     bool
 */
public function prepare($query) {}

/**
 * Bind a value to a named or question mark placeholder
 * in the prepared SQL statement.
 *
 *  @param      mixed   The parameter identifier. For named placeholder, this value must be a
 *                      string (:name). For a question mark placeholder, the value must be the
 *                      1-indexed position of the parameter.
 *  @param      mixed   The value to bind to the parameter.
 *  @param      int     Data type for the parameter, using the predefined PDO constants:
 *                      http://php.net/manual/en/pdo.constants.php
 *  @return     bool
 */
public function bindValue($param, $value, $dataType) {}

/**
 *  Bind a referenced variable to a named or question mark
 *  placeholder in the prepared SQL statement.
 *
 *  @param      mixed   The parameter identifier. For named placeholder, this value must be a
 *                      string (:name). For a question mark placeholder, the value must be the
 *                      1-indexed position of the parameter.
 *  @param      mixed   Variable to bind to the parameter.
 *  @param      int     Data type for the parameter, using the predefined PDO constants:
 *                      http://php.net/manual/en/pdo.constants.php
 *  @return     bool
 */
public function bindParam($param, &$variable, $dataType) {}

/**
 *  Number of rows affected by last operation.
 *
 *  @return     int
 */
public function rowCount() {}

/**
 *  Return the id of last inserted row.
 *
 *  @return     int
 */
public function lastInsertId() {}

}
```
### Examples
Retrieving from the database:
```php
// Prepare the query
$database->prepare("SELECT SomeColumn FROM SomeTable");

// Call the shorthand method read() to fetch the results
$response = $database->read();

// Because the read/write methods prepares the statement, the above code can be shortened:
$response = $database->read("SELECT SomeColumn FROM SomeTable");

// Retrieve a specific row like this:
$response = $database->read("SELECT SomeColumn FROM SomeTable WHERE id = :id", array("id" => $id));
```
Adding to the database
```php
$values = array(
  "ThisValue"  => "Foo!",
  "ThatValue"  => "Bar!"
);

$SQLQuery = "INSERT INTO SomeTable (someColumn, anotherOne) VALUES (:ThisValue, :ThatValue)";
$database->prepare($SQLQuery);

// Bind the values that are to be inserted
$database->bindValue(":ThisValue", $values["ThisValue"]);
$database->bindValue(":ThatValue", $values["ThatValue"]);

$response = $this->write();
```
Deleting from the database
```php
$SQLQuery = "DELETE FROM SomeTable WHERE id = :id";
$SQLParam = array("id" => $id);
$response = $database->write($SQLQuery, $SQLParam);
```
A bit more complicated example:
```php
$values = array(
  array(
    "ThisValue"  => "Foo 123",
    "ThatValue"  => "Bar 456"
  ),
  array(
    "ThisValue"  => "Foo 789",
    "ThatValue"  => "Bar 012"
  ),
  array(
    "ThisValue"  => "Foo 345",
    "ThatValue"  => "Bar 678"
  )

// Create the unnamed placeholders, based on the number of values the row will take:
$markers = array_fill(0, count($values[0]), '?');
$markers = '(' . implode(", ", $markers) . ')';

// The number of placeholders must match the number of values that are to be inserted 
// in the VALUES-clause. Create the array with array_fill() and join the array with 
// the query-string.
$clause = array_fill(0, count($values), $markers);
$query  = "INSERT INTO SomeTable (someColumn, anotherOne) VALUES " . implode(", ", $clause);
$database->prepare($query);

// Bind the values using bindValue(). Using question marked placeholders, the value 
// must be 1-indexed, that is starting at position 1.
$index = 1;
foreach ($values AS $key => $value) {
  $this->bindValue($index++, $value['ThisValue']);
  $this->bindValue($index++, $value['ThatValue']);
}

// A more pretty and dynamic way to write the above statement could be by going
// by the columns of the array, like so:
$columns = array_keys($values[0]);
foreach ($values AS $key => $value) {
  foreach ($columns AS $column)
    $this->bindValue($position++, $value[$column]);
  }
}

// And don't forget to write to database
$response = $database->write();
```
### Author
Database was written by Ardalan Samimi.
