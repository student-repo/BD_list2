1)

create table if not exists book(AuthorFirstName varchar(30), AuthorLastName varchar(30), ISBN int (13), title varchar(100));

create table if not exists reader(FirstName varchar(30), LastName varchar(30), BirthDate datetime, PIN int (11));

create table if not exists library_brunch(Name varchar(50), Address varchar(50));

2)

create table if not exists book(BookID int(5) not null auto_increment, AuthorFirstName varchar(30), AuthorLastName varchar(30), ISBN int (13) unique, title varchar(100), primary key(BookID));


create table if not exists reader(ReaderID int(5) not null auto_increment, FirstName varchar(30), LastName varchar(30), BirthDate datetime, PIN int(11) unique, primary key(ReaderID));


create table if not exists library_brunch(LibraryBrunchID int(5) not null auto_increment, Name varchar(50), Address varchar(50), primary key(LibraryBrunchID));


3)

create table if not exists borrowed(BorrowID int(5) not null auto_increment, ReaderID int not null, BookID int not null, BorrowDate datetime not null default now(), primary key(BorrowID), constraint `reader_key` foreign key (ReaderID) references reader(ReaderID), constraint `book_key` foreign key (BookID) references book(BookID));
