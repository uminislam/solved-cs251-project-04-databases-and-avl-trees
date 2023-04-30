Download Link: https://assignmentchef.com/product/solved-cs251-project-04-databases-and-avl-trees
<br>
Conceptually, a <strong>database</strong> is a set of <strong>tables</strong>, where each table stores data that is logically related.  For example, the database maintained by UIC would include the following tables:

<strong>students</strong>:  data about each student <strong>faculty</strong>:  data about each faculty member <strong>staff</strong>:  data about each staff employee <strong>courses</strong>:  data about each course

Databases are created, searched and modified using the <strong>SQL</strong> programming language.  For example, here’s an SQL select query to retrieve all the info from the students table about the student with netid fzizza2:

<strong>select * from students where netid = fzizza2 </strong>

How are databases implemented?  Each table is stored in a file, and balanced trees are built to index the file and find data quickly.  For example, suppose student fzizza2 has their data stored at position 2048 in the underlying file (which would be called “students.data” based on the tablename).  This file would be indexed by a tree consisting of (uin, offset) pairs, and this student’s node in the tree would contain the pair (fzizza2, 2048).

<h1>Your Assignment</h1>

Your assignment is to build a database-like program that inputs SQL select queries, and retrieves the requested data.  Your program will build and use trees to find data that is indexed, and otherwise use linear search to find data that is not indexed.  Here’s an example interaction with the program:

In this case the underlying “students” table has 2 indexes, based on columns <strong>uin</strong> and <strong>netid</strong>.  Searches based on those columns are fast since they will take advantage of an underlying AVL tree; examples are select queries #1 and #2 shown above.  However, select query #3 searches based on <strong>lastname</strong>, which is not indexed.  This will require a linear search of the underlying “students.data” file.




<h1>File formats</h1>

File formats are different from what you might expect, so please read this carefully.  A <strong>table</strong> is represented using 2 files:  one that contains meta-data (information about the table), and one that contains the data itself.  For simplicity, we are going to use text files, so you can open both files in a text editor and review the contents.  All data will be string-based, again for simplicity.  Let’s look at the <strong>students</strong> example.

For the <strong>students</strong> table, the meta-data file will be called “students.meta”, and the data file “students.data”.  Even though these are text files, note that you cannot double-click on the files to open — since the extensions are .meta and .data, and not .txt.  To open these files, you’ll need to start your text editor, and then open from within the text editor program itself.




First, let’s look at the data stored in “students.data”.  We are storing 5 values per student:  uin, first name, last name, netid, and email address:




123456 pooja sarkar psarka12 <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="34445b5b5e5574445d554e4e551a575b59">[email protected]</a> …………………………….

456789 frank zizza fzizza2 <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="b6ccdfccccd7f6c3d5dad798d3d2c3">[email protected]</a> ………………………………..

789123 mee kim mkim16 <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="a3cec8cace9295e3d6cac08dc6c7d6">[email protected]</a> …………………………………….

238117 lucy johnson ljohns21 <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="432f36203a6d292c2b2d302c2d03242e222a2f6d202c2e">[email protected]</a> ……………………….

556178 drago kolar dkolar8 <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="d5b1a7b4b2bae4ececed95acb4bdbabafbb6bab8">[email protected]</a> …………………………… 732290 soo kim skim21 <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="90e3fbf9fda2a1d0e5f9f3bef5f4e5">[email protected]</a> …………………………………….




The “.” can be ignored; they are used to visually pad the line so that every student “record” in the file is the same size.  All data values are treated as strings, even though they may look like integers (e.g. uin).  The .data file is in Windows format, and must remain in Windows format (with r and 
 at the end of each line); do not convert these files to your local text file format.




Looking at the data as a whole, each kind of data — uin, first name, last name, etc. — is viewed as a <strong>column</strong> of data.  This implies the students table contains 5 columns.  Where are the names of these columns defined?  Which columns are indexed to enable fast lookup?  This is the type of “meta-data” stored in the 2<sup>nd</sup> file, “students.meta”:




82 5 uin 1 firstname 0 lastname 0 netid 1 email 0




The first line is the <strong>record size</strong> <strong>RS</strong>, and denotes the amount of data (in bytes) stored per student.  This implies the first student will be stored at offset 0, the 2<sup>nd</sup> student at offset 82, the 3<sup>rd</sup> student at 164, and so on.  In general, record offsets are a multiple of RS.  The second line is the <strong># of columns</strong> <strong>C</strong> in the table, and corresponds to the number of data values stored per record.  For example, each student record contains 5

data values (which matches what we saw earlier in “students.data”).  Lastly, the remainder of the .meta file contains C lines, one per column, defining the name of the column along with whether this column is indexed (1) or not (0).  Given the “students.meta” file shown above, we know the students table has 5 columns — uin, firstname, lastname, netid, email — where <strong>uin</strong> and <strong>netid</strong> are indexed.  Assume that the order of the column names in .meta file matches the order of the data in each record of the .data file.  In other words, in “students.data”, the 1<sup>st</sup> data value is the uin, the 2<sup>nd</sup> data value is the firstname, the 3<sup>rd</sup> data value is the lastname, and so on.




In terms of C++, the .meta file can be opened and processed as you might expect:  use an <strong>ifstream</strong> object, call <strong>file.good( )</strong> to make sure it was opened successfully, and input the data using the <strong>&gt;&gt;</strong> operator.  In general, since the # of columns is unpredictable, you’ll want to save the meta-data in a data structure such as vector.

The .data file requires different processing because it’s based on fixed-sized records.  The use of fixed-sized records is necessary so that data for a particular student can be found quickly — by jumping to a particular offset (versus reading the file line by line).  You open the file as a <strong>binary file</strong>, and you move from line to line by moving from the start of one record to the start of the next.  Here’s an example of opening a .data file (such as “students.data”) and echoing the values stored in each record:




<em>#include &lt;iostream&gt; </em>

<em>#include &lt;fstream&gt; </em>

<em>#include &lt;vector&gt; </em>

<em>#include &lt;string&gt; </em>

<em>#include &lt;sstream&gt; </em>

<em> </em>

<em>using namespace std; </em>




void <strong>EchoData</strong>(string tablename, int recordSize, int numColumns)

{

<strong>  string   filename = tablename + “.data”;   ifstream data(filename, ios::in | ios::binary); </strong>

<strong>   if (!data.good()) </strong>

<strong>  { </strong>

<strong>     cout &lt;&lt; “**Error: couldn’t open data file ‘” &lt;&lt; filename &lt;&lt; “‘.” &lt;&lt; endl;      return; </strong>

<strong>  } </strong>




//

// Okay, read file record by record, and output each record of values:

// <strong>  data.seekg(0, data.end);  </strong>// move to the end to get length of file:<strong>   streamoff length = data.tellg(); </strong>

<strong>   streamoff pos = 0;  </strong>// first record at offset 0: <strong>  string    value; </strong>

<strong> </strong>

<strong>  while (pos &lt; length) </strong>

<strong>  {      data.seekg(pos, data.beg);  </strong>// move to start of record:

<strong>      for (int i = 0; i &lt; numColumns; ++i)  </strong>// read values, one per column:

<strong>     { </strong>

<strong>       data &gt;&gt; value;        cout &lt;&lt; value &lt;&lt; ” “; </strong>

<strong>     }       cout &lt;&lt; endl;      pos += recordSize;  </strong>// move offset to start of next record: <strong>  } </strong>

<strong>} </strong>







To echo the data in “students.data”, you would call as follows:  <strong>EchoData(“students”, 82, 5);</strong>




<h1>Assignment details</h1>

In a database, the data stays in the .data file.  Trees are used to find the data quickly, but the (key, value) pairs stored in the tree are of the form (key, record offset) — not (key, data).  When the program starts, you’ll read the .meta file and determine which columns of data are indexed.  For each index, you’ll build a tree.  How is the tree built?  You open the .data file, read through the data record by record, and insert into the tree the <strong>(key, offset)</strong> pair for that record.  You don’t insert the data into the tree, you insert the record positions so you can find the data later.  This is how databases work, and how you must approach this assignment.




For example, the <strong>students</strong> meta-data states that the <strong>uin</strong> and <strong>netid</strong> columns are indexed.  Using the “students.data” file we saw earlier:




123456 pooja sarkar psarka12 <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="75051a1a1f1435051c140f0f145b161a18">[email protected]</a> …………………………….

456789 frank zizza fzizza2 <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="6f150615150e2f1a0c030e410a0b1a">[email protected]</a> ……………………………….. 789123 mee kim mkim16 <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="660b0d0f0b575026130f0548030213">[email protected]</a> ……………………………………. .

.




To build the <strong>uin</strong> index tree, you would insert (“123456”, 0), then (“456789”, 82), then (“789123”, 164), etc.  The offsets are multiples of the record size, which is 82 in this case.  To build the <strong>netid</strong> index tree, you would insert (“psarka12”, 0), then (“fzizza2”, 82), then (“mkim16”, 164), etc.  The keys are strings, and the record offsets are type <strong>streamoff</strong> defined in #include &lt;fstream&gt;.  This implies your trees should be of type avltree&lt;string, streamoff&gt;.




One of the more interesting aspects of the assignment is that you need to use other data structures.  There can be any number of columns, and any number of indexes.  So you’ll need to keep track of the columns, and your index trees, using a data structure.  [ <em><u>Hint</u>:  std::vector is a good choice.</em> ]




Use functions.  For example, I found the following function very helpful for reading a record from the .data file.  Here’s the design, we’ll leave the coding details to you:




//

// GetRecord

//

// Reads a record of data values and returns these values in a vector.

// Pass the table name, the file position (a stream offset), and the #  // of columns per record.

//

// Example:  GetRecord(“students”, 0, 5) would read the first student

// record in “students.data”.

//

vector&lt;string&gt; <strong>GetRecord</strong>(string tablename, streamoff pos, int numcolumns) {

// open the file, make sure it opened, seekg to the given position,    // loop and input values using &gt;&gt;, store into vector, return vector }




This function opens and closes the file every time; this may not be the most efficient approach, but it’s a good place to start.  If time permits, you can improve efficiency by opening the .data file once in main(), leaving it open, and passing the file object instead of the tablename; you’ll need to pass the file object by reference.




Use the avltree class you built in project #03; a compiled avl.o object file will be provided on Codio if you were unable to complete project #03.  However, do not change the tree in any significant ways, the current plan for testing submissions is to run your database program using our avltree class so we can report tree usage statistics; this implies you cannot change the public API.  That said, we ran into some compilation errors around the avltree class when returning trees from a function, so you might need to modify the copy constructor declaration by adding the <strong>const</strong> keyword as shown below:




// copy constructor:

avltree(<strong>const</strong> avltree&amp; other)

{

Root = nullptr;

Size = 0;




_copytree(other.Root);

}




Likewise, in some cases you might want to assign a tree into another variable, which can also happen when returning a tree from a function.  In this case you need to overload the assignment operator <strong>=</strong>, like this:




<strong>avltree&amp; operator=(const avltree&amp; other) </strong>

{

clear();




_copytree(other.Root);




return *this;

}







<h1>Programming Environment and Getting Started</h1>

You are free to program in any environment you want; final submissions will be collected using Gradescope.  If you want to use Codio, a project has been created named “<strong>cs251-project04-myDB</strong>”.  The environment will contain catch and valgrind for testing purposes.




A skeleton “main.cpp” file is provided that contains the EchoData function shown earlier, as well as some code that implements the main loop.  Type “<strong>make build</strong>” to compile, and “<strong>make run</strong>” to run.  Sample input files are provided as well.




The makefile is setup to compile and run against our implementation of the avltree class.  If you would prefer to use your own avltree class, edit the makefile and remove all references to “avl.o”.  Then delete the “avl.o” and “avl.h” files, and install your own version of “avl.h”.







<h1>Requirements</h1>

Design and build a database program as outlined in the previous sections.  You must build an AVL tree for each indexed column, and use that tree for fast lookup when the select query’s <strong>where</strong> clause references that column.  Here’s another screenshot — the <strong>uin</strong> index tree must be used in processing queries #1 and #2:




For queries #3 and #4 shown above, linear search of the underlying .data file must be performed since <strong>firstname</strong> is not an indexed column.




Reasonable error checking should be performed, in particular around the user’s input.  If the user enters a query other than select or exit, it’s an error.  A valid select query has 8 components.  Version 1 selects a particular column of data, given as columnname1 in the query below:




<strong>select</strong> columnname1 <strong>from</strong> tablename <strong>where</strong> columnname2 = value




Version 2 selects all columns of data by replacing columnname1 with *:




<strong>select</strong> * <strong>from</strong> tablename <strong>where</strong> columnname2 = value

For ease of input processing, assume all query components are separated by a single space.  Here’s a screenshot of some error handling cases: