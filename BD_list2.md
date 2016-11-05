1)

create table if not exists book(AuthorFirstName varchar(30), AuthorLastName varchar(30), ISBN int (13), title varchar(100));

create table if not exists reader(FirstName varchar(30), LastName varchar(30), BirthDate datetime, PIN int (11));

create table if not exists library_brunch(Name varchar(50), Address varchar(50));

2)

create table if not exists book(BookID int(5) not null auto_increment, AuthorFirstName varchar(30), AuthorLastName varchar(30), ISBN int (13) unique, title varchar(100), primary key(BookID));


create table if not exists reader(ReaderID int(5) not null auto_increment, FirstName varchar(30), LastName varchar(30), BirthDate datetime, PIN int(11) unique, primary key(ReaderID));


create table if not exists library_brunch(LibraryBrunchID int(5) not null auto_increment, Name varchar(50), Address varchar(50), primary key(LibraryBrunchID));


3)

  create table if not exists borrowed(BorrowID int(5) not null auto_increment, ReaderID int not null, BookID int not null, BorrowDate datetime not null default now(), primary key(BorrowID), constraint `reader_key` foreign key (ReaderID) references reader(ReaderID) on delete cascade, constraint `book_key` foreign key (BookID) references book(BookID) on delete cascade);


4)


delimiter $$
create procedure foo10()
wholeblock:begin
declare x int;
set x=1;
loop_label: loop
if x>100 then
leave loop_label;
end if;
if x < 51 then
insert into reader(FirstName, LastName, BirthDate, PIN) values (concat("FirstName", x), concat("LastName", x), date("1990-01-01") + interval x*x day, x);
end if;
if x<6 then
insert into library_brunch(Name, Address) values (concat("Name", x), concat("Address",x));
end if;
insert into book(AuthorFirstName, AuthorLastName, ISBN, title) values (concat("AuthorFirstName", x), concat("AuthorLastName",x), x, concat("title",x));
set x=x+1;
iterate loop_label;
end loop;
end $$


after that delimiter ;
and call foo9();

delimiter $$
create procedure foo11()
begin
declare x int;
set x=1;
while x <= 100 do
  if x <= 50 then
    insert into reader(FirstName, LastName, BirthDate, PIN) values (concat("FirstName", x), concat("LastName", x), date("1990-01-01") + interval x*x day, x);
  end if;

  if x<=5 then
    insert into library_brunch(Name, Address) values (concat("Name", x), concat("Address",x));
  end if;

  insert into book(AuthorFirstName, AuthorLastName, ISBN, title) values (concat("AuthorFirstName", x), concat("AuthorLastName",x), x, concat("title",x));
  set x=x+1;
end while;
end $$
delimiter ;

5)

delimiter $$
create procedure foo21()
begin
declare x int;
declare y int;
set x=1;
set y=1;

while x <= 50 do
  set y = 1;

  while y <= 30 do
      insert into borrowed(ReaderID, BookID, BorrowDate) values (x,x + y , now() - interval y day);
      set y = y + 1;
   end while ;

set x = x + 1 ;
end while;

end $$
delimiter ;


6)

create table if not exists book_in_branch(LibraryBrunchID int(5), BookID int(5), LaunchDate datetime not null, constraint `library_brunch_key` foreign key (LibraryBrunchID) references library_brunch(LibraryBrunchID) on delete cascade, constraint `book_key2` foreign key (BookID) references book(BookID) on delete cascade);



create table if not exists book_in_branch(LibraryBrunchID int(5), BookID int(5), LaunchDate datetime not null);

7)

insert into library_brunch(Name, Address) values ("Name6", "Address6");

8)

delimiter $$
create trigger insert_book
  after insert on book
  for each row
begin
  insert into book_in_branch (LibraryBrunchID, BookID, LaunchDate) values (6,new.BookID,now());
end $$
delimiter ;

if you want to check:  insert into book(AuthorFirstName, AuthorLastName, ISBN, title) values ("AuthorFirstName", "AuthorLastName",101, "title");
9)

delimiter $$
create trigger delete_book_in_branch
  after delete on book_in_branch
  for each row
begin
  delete from book where BookID=old.BookID;
end $$
delimiter ;

10)

delimiter $$
create trigger foo
  after update on reader
  for each row
begin
  if (old.LastName != new.LastName) then
    delete from borrowed where ReaderID=old.ReaderID;
  end if;
end $$
delimiter ;


if you want check: update reader set LastName="Ala ma kota" where ReaderID=30;


11)

delimiter $$
create procedure foo1(
  in readerLastName varchar(30),
  out total int)
  begin
  select count(*)
  into total
  from borrowed where ReaderID in (select ReaderID from reader where LastName=readerLastName);
  end $$
  delimiter ;

  if you want to check: call foo1("LastName25", @total);
  select @total;

  12)

  delimiter $$
  create procedure foo2(
    in PIN int(11))
    begin
    select title from borrowed inner join reader on borrowed.ReaderID=reader.ReaderID inner join book on borrowed.BookID=book.BookID where reader.PIN=PIN;
    end $$
    delimiter ;

if you want to check: call foo2(11);


13)

create view foo5 as
select title, Name, concat(FirstName, " ", LastName) from borrowed inner join reader on borrowed.ReaderID=reader.ReaderID inner join book on book.BookID=borrowed.BookID left outer join book_in_branch on book_in_branch.BookID=borrowed.BookID left outer join library_brunch on book_in_branch.LibraryBrunchID=library_brunch.LibraryBrunchID;

or
create view foo5 as
select title,LibraryBrunchID , concat(FirstName, " ", LastName) from borrowed inner join reader on borrowed.ReaderID=reader.ReaderID inner join book on book.BookID=borrowed.BookID left outer join book_in_branch on book_in_branch.BookID=borrowed.BookID;


if you want to check: select * from foo5;

14)

delimiter $$
create procedure foo3(
  in name varchar(50))
  begin
  select count(*) from book_in_branch inner join library_brunch on book_in_branch.LibraryBrunchID=library_brunch.LibraryBrunchID where library_brunch.Name=name;
  end $$
  delimiter ;


15)

delimiter $$
create procedure foo4(
  in libraryBrunchName varchar(50),
  out youngestBook varchar(50),
  out oldestBook varchar(50))
  begin
  select title
  into youngestBook
  from book_in_branch
  inner join book on book_in_branch.BookID=book.BookID
  inner join library_brunch on library_brunch.LibraryBrunchID=book_in_branch.LibraryBrunchID
  where library_brunch.Name=libraryBrunchName
  order by LaunchDate desc limit 1;

  select title
  into oldestBook
  from book_in_branch
  inner join book on book_in_branch.BookID=book.BookID
  inner join library_brunch on library_brunch.LibraryBrunchID=book_in_branch.LibraryBrunchID
  where library_brunch.Name=libraryBrunchName
  order by LaunchDate limit 1;

  end $$
  delimiter ;

  if you want to check: call foo4("Name6", @young, @old);
  select @young;
  select @old;
