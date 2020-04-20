Escaping References
===================
Java provided Encapsulation feature where the data is completely encapsulated in a class and behaviors/methods
determine how that data can be accessed by other classes.
This Encapsulation control is achieved by access modifier keywords like private,default,protected,public.
Even though we define the members of the class as private there are some situations where References are escaped.
Eg:

public Class Customer {
  private String name;

  public Customer(String name) {
    this.name=name;
  }

  public String getName() {
    return name;
  }

  public String toString() {
    return name;
  }

}
If we see the customer class it is completely Immutable, once the object is instantiated no one can change its state as there
is no setter method to modify its state.

public Class CustomerRecords {
  private Map<String,Customer> records;

  public CustomerRecords() {
    this.records=new HashMap<>();
  }

  public void addCustomer(Customer c) {
    records.put(c.getName(),c);
  }

  public Map<String,Customer> getCustomers() {
    return records;
  }
}

As you can see above CustomerRecords class is NOT completely Immutable even though records variable is private.
This is because we can get a Reference of this records map and modify its content which we don't want as per the requirement,
and this reference we will call as Escaping Reference.

public static void main(String[] args) {
  CustomerRecords customerRecords = new CustomerRecords();
  customerRecords.addCustomer(new Customer("leela"));
  customerRecords.addCustomer(new Customer("prasad"));

  Map<String,Customer> records = customerRecords.getCustomers();
  //This records I can do anything, which will corrupt the original records.
  records.clear();
}

Solutions
=========
1.
To avoid this we can return a new collection every time when getCustomers() method is called.
public Map<String,Customer> getCustomers() {
  return new HashMap<String,Customer>(records);
}

But this will be create confusion in the client code, if we don't know that getCustomers() call is returning a new Map containing
records.

public static void main(String[] args) {
  CustomerRecords customerRecords = new CustomerRecords();
  customerRecords.addCustomer(new Customer("leela"));
  customerRecords.addCustomer(new Customer("prasad"));

  Map<String,Customer> records = customerRecords.getCustomers();

  records.clear();

  for(Customer cust : records.values()) {
    println(cust);
  }
}

println(cust) line will still print the records even though
line records.clear() is executed, this is because in the CustomerRecords class they are returning new Map for every call.
and this will make less readable.

2.
To avoid readability issue we will return an unmodifiableCollection, so that client cannot modify the collection.

public Map<String,Customer> getCustomers() {
  return Collections.unmodifiableMap(records);
}

public static void main(String[] args) {
  CustomerRecords customerRecords = new CustomerRecords();
  Map<String,Customer> records = customerRecords.getCustomers();

  records.clear(); // This will throw UnsupportedOperationException as client is trying to modify the collection.
}

======
How to create Immutable custom Objects like Customer
public Class Customer {
  private String name;

  public Customer(String name) {
    this.name=name;
  }

  public String getName() {
    return name;
  }

  public void setName() {
    this.name=name;
  }

  public String toString() {
    return name;
  }

}

public Class CustomerRecords {
  private Map<String,Customer> records;

  public CustomerRecords() {
    this.records=new HashMap<>();
  }

  public void addCustomer(Customer c) {
    records.put(c.getName(),c);
  }

  public Map<String,Customer> getCustomers() {
    return Collections.unmodifiableMap(records);
  }

  public Customer getCustomerByName(String name) {
    return records.get(name);
  }
}

If you see the method getCustomerByName(name) will return the Customer class.
and we made this Customer class Mutable by adding setName(name) method.
so any client can mutate Customer class state.

public static void main(String[] args) {
  CustomerRecords customerRecords = new CustomerRecords();
  customerRecords.addCustomer(new Customer("leela"));
  customerRecords.addCustomer(new Customer("prasad"));

  Customer customer = customerRecords.getCustomerByName("prasad");
  customer.setName("jagu");
}

Solutions:
==========
1. By creating a copy constructor so that client will get a new object every time when getCustomerByName() is accessed.

public Class Customer {
  private String name;

  public Customer(String name) {
    this.name=name;
  }

  //copy constructor
  public Customer(Customer customer) {
    this.name=customer.name;
  }

  public String getName() {
    return name;
  }

  public void setName() {
    this.name=name;
  }

  public String toString() {
    return name;
  }

}

public Class CustomerRecords {
  private Map<String,Customer> records;

  public CustomerRecords() {
    this.records=new HashMap<>();
  }

  public void addCustomer(Customer c) {
    records.put(c.getName(),c);
  }

  public Map<String,Customer> getCustomers() {
    return Collections.unmodifiableMap(records);
  }

  public Customer getCustomerByName(String name) {
    return Customer(records.get(name)); //copy constructor is called to get new customer object.
  }
}

2. By declaring Interfaces for ReadOnlyMethods are return this Interface to the client.

public interface CustomerReadOnly {

  public abstract getName();
  public abstract toString();
}

public Class Customer implements CustomerReadOnly {
  private String name;

  public Customer(String name) {
    this.name=name;
  }

  @Override
  public String getName() {
    return name;
  }

  public void setName() {
    this.name=name;
  }

  @Override
  public String toString() {
    return name;
  }

}

public Class CustomerRecords {
  private Map<String,Customer> records;

  public CustomerRecords() {
    this.records=new HashMap<>();
  }

  public void addCustomer(Customer c) {
    records.put(c.getName(),c);
  }

  public Map<String,Customer> getCustomers() {
    return Collections.unmodifiableMap(records);
  }

  public CustomerReadOnly getCustomerByName(String name) {
    return records.get(name);
  }
}




Key Takeaways.
1. In get methods or read methods we need to check whether returned type is mutable or not
if it is mutable then we need to try to make that class as immutable,if that is not possible then
we will return a new object using copy constructor, because using readonly interfaces will give a
chance to clients to cast to the specific class and mutate.
2. In any class if have a method(eg: setter method) which will mutate the state then we need to
check in the entire project for the objects of that type, whether they are returning new objects of that
type using copy constructor other wise it will affect other methods.
3. Mutation means changing the state of the class.
If the class has no state(i.e., no member variables) then the class is by default immutable.
stateless classes are always Immutable.
***** Every Get Method/Read Method should not change the state, as it will create inconsistency.


Garbage Collection
==================

String intern concept:
======================
intern method in string class results in creation of string in String pool.

String one = "hello";
String two = "hello";
one == two //equal

String three = new Integer(75).toString();
String four = "75";
three == four //NOT equal, as strings which are formed during calculation will not
              //be created in string pool.

so this time we will make the JVM to forcely create on string pool.
tring three = new Integer(75).toString().intern();
String four = "75"; //Equal, as intern() created 75 on string pool.


Object creation in Java Language.
=================================
In Java Objects are created in Heap memory only.
but this statement not completely true, because in Modern JVMs will make smart decisions
whether the object created on stack or heap.
Modern JVM analyse whether the object is reusable across block codes or not,if that is not
reusable then JVM will create in stack.
But in General 90% of objects are created in Heap memory only.


Garbage Collection
Any Object on the Heap which cannot be reached through a reference from the stack
is eligible for Garbage collection.

We can Run Garbage collection from our application using System class
System.gc()
When Garbage Collection is running it will stop all the threads in the application and resumes only when Garbage collection
process is completed, for this reason GC process should be quick and infrequent.

finalize() method will be called by the Garbage collector on an object when the gc process is running and that object is
eligible for Garbage Collection.
Note that sometimes GC will not clear all unreachable objects in the heap area, so when finalize method will be called is
not Guaranteed by the JVM.
If the object is always reachable from stack then finalize method will never run.
Therefore it is not advisable to do cleanup code in finalize method.
One case we can use finalize method is checking whether any resources are open during destruction of object so that we
can detect memory leaks from the logs.

public void finalize() {
  if(file.isOpen()) {
    log.warn("Looks like File Resource is Not closed")
  }
}


Soft Leak: An object is referenced on the stack even though it will never be used again.


Garbage Collection Process:
GC Follows Mark and Sweep Algorithm

Mark Stage:
1. In the Marking stage first it will stop all the threads running in the application, because GC cannot work properly
if any threads are running in the application.
2. After stoping all threads the GC will mark all the objects which are reachable from stack.

Sweep Algorithm:
Now it will scan entire heap and remove objects which are not marked in the above process.

GC will not collect the Garbage instead it will mark the live references and sweep all the objects which are not
reachable from stack. So GC will be much faster if there is more garbage because only less objects will be eligible
for marking.

GC Process should be faster enough as it stops all threads in the application. To increase performance of GC process
It will divide entire Heap into 2 parts.
1. Young Generation.
2. Old Generation.

Young Generation is again divided in to 3 parts
1. Eden space
2. Surviour0 (S0)
3. Surviour1 (S1)

Most of the times objects created by Application will live for a shorter period. Due to this reason newly
created objects are first placed in Eden space. If any object survived for 8 GC Cycles in Young Generation Heap space
they are likely to exist for a longer duration. So after 8 GC Cycles those objects are moved to Old Generation.
And Old Generation GC will happen very rarely. In this way GC Operation is optimized.

cycles in Young Generation:
1st cycle : Eden Space to S0
2nd cycle : Eden Space to S1 and S0 to S1
3rd cycle : Eden Space to S0 and S1 to S0
4th cycle : Eden SPace to S1 and S0 to S1
.
.
.
After 8 cycles Eden Space + S0 or S1 space will be moved to Old Generation.
These 8 Cycles is not fixed and JVM will vary this number based on the JVM memory allocated to the Application.

During out of heap space issue Generally Eden space cannot be moved to Old Generation, so it will not move the objects to S0 or S1
that is the reason why we have difference in the total heap and the max heap used by the system during out of heap space issue.

PermGen/Metaspace
=================
PermGen means Permanent Generation.
NO Garbage Collection will happen for PermGen. So Application will Crash if PermGen Space is FULL.
There are 2 types of objects will be stored in PermGen.
1. Internalized Strings i.e., string which are created using intern() method.
2. Classes Metadata i.e., classes will be loaded/unloaded automatically when it is required.
  Classes Metadata will be stored in this PermGen
Upto Java 1.6 PermGen is part of Heap Memory.
In Java 1.7 Internalized Strings is removed as part of PermGen Region, so only classes Metadata will be stored in PermGen.
In Java 1.8 PermGen is REMOVED from Heap Area and is separately Allocated. We should be able to control the max PermGen that
can be allocated in the application otherwise it may take entire RAM space if it need to grow.


Heap Size
=========
Default Heap Size(when you are running as a server) allocated will 1/4 of RAM Size.
On a client Heap size will be 256MB.

1. Setting Max Heap Size -Xmx
eg: -Xmx256m or -Xmx204800k or -Xmx1g

2. Setting Min Hep Size -Xms
eg: -Xms40m
if it needs more that 40MB then JVM automatically increase the heap size on the fly, but it will have performance
implications for mission critical systems, so it is better to give a resonable min heap size.

3. Setting PermGen/Metaspace
-XX:MaxPermSize
eg: -XX:MaxPermSize=256m

4. Printing Logs when GC Happens.
-verbose:gc

5. Setting Young Generation size -Xmn
eg: -Xmn256m
Note that this is the Young generation Min and Max Heap Size allocated at startup,
so our Min Heap Size for our application should be >256m i.e., -Xms>256m so that it can allocate 256m to Young Generation.
This setting especially useful when we have more short lived objects that can survive 8 GC cycles.
Recommended Size is between Quarter and half of jvm heap size.

6. Creating Heap Dump file when Memory Out of Exception Happens
-XX:HeapDumpOnOutOfMemory


GC Collectors:
==============
There are 2 types of Collectors
1. Serial
  -XX:+UseSerialGC
2. Parallel
  -XX:+UseParallelGC
