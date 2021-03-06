#Appendix lglib 

##Introduction to lglib
lglib is a cornerstone for the lua-based web framework [bamboo](https://github.com/daogangtang/bamboo). Generally speaking, it provides a high-level API for customized use on the following aspects:   

+ Extension of several lua standard libraries, like string, table and io;   
+ Add several data structures for use, i.e., Dict, Set, List, Queue, etc;  
+ Implementation of Object-Oriented Programming mechanism (OOP);     
+ Global helper functions, like http-related methods.       

After injecting methods of lglib.string, lglib.table, lglib.io into namespaces of lua standard libraries, string, table and io separately, the three lua standard libraries could be used much easier and powerful. For example, `io.loadFile()` can load source file into memory directly, `table.deepCopy()` could make an exact copy of one table even with nested structures, and much more. In later sections, we will discuss them in details.

Based on the extended table library, several data structures with high-level API, including Dict, Set, List, Queue and extra sorted versions, have been implemented. At the same time, OOP mechanism is also implemented on the top of the powerful lua-table. Now we introduce several global APIs for these two parts:

>>List: list API should be splited into list and queue/stack further     


`setProto()` is just to config a prototype for a given object/instance.  `T()` is a special case of `setProto()` that its prototype is always lua-table. For example, `setProto(obj, proto/table)`means `obj` could access methods and fields of instance `proto/table` for reuse. Object is the rooted parent for all lua class instances, you can use it to define a new class as following:		

	local Message = Object:extend{....}
		__name = 'Message';  
		__desc = 'General message definition.';  
	  
		-- constructor
		init = function (self, t)  
		    if not t then return self end     
	  
		    self.type = t.type  
		    self.author = t.author  
		    self.content = t.content  
		    self.timestamp = t.timestamp or os.time()  
	  
		    return self  
		end;  
	  
		doSomething = function (self)  
		    .....  
		end;
	}  
  

       		
`typename()` returns the field of `__typename` of data structures, like List, Dict, Queue and Set. More conveniently, methods like `isList()`, `isDict()` or `isSet()` can check whether an object is an instance of List/Dict/Set or not. For basic data types that lua provided, `checkType()` is employed for assertion, like    
	
	checkType(a, b, c, 'string', 'table', 'number')   
	 		
passes only if a is string, b for table and c for nubmber.

Finally, some helper functions are explained here. `checkRange()`can be used as `checkRange(a, 0, 10, b, 20, 30, c, 10, 100)` for range assertion. `isFalse()` will treat  any of `nil`, `false`, `0`, `""` and`{}` as false for conditional testing. Some io-related methods follows as: `printf()` as an alias of `string.format()` can be used in a C-printf way. `ptable()` only prints one-layer of table information, while `fptable()` printing all details of a table even with nested structures. `serialize(obj)` serializes lua instances/objects into a long string but in a lua-table format, while `deserialize(obj_str)` as deserialization just loading serialized string into memory. And the point is that format of serialzation are just lua code of assignment of lua-tables. 


## Extension of lua standard libraries
###String    
Definitions of some magic methods `__mul`, `__mod`, `__div` and `__add` are added or modified. Let'd explain them from the simper one, `__add`. This definition of `__add` is overrided, and `a + b` is the same as `a .. b`. `__mul(formatstr, str/listOfStr)` just formatting one string or list of string.  `__mod` and `__div` are special wrappers of string substitutions. For example,      
	
	print( "${name} is ${value}" % {name = "foo", value = "bar"} ) 				----> "foo is bar"
	print( "%(key)s is %(val)7.2f%" / {key = "concentration", val = 56.2795} )  ----> "concentration is   56.28%"
	
string can be treated as an array of characters. Therefore, random accessing and slicing operations are also better APIs for lua-string. They are `index(self, i)` that selecting a letter from a given string and `slice(self, i, j)` that selecting a continous piece of letters from a given string.


Some pattern matching and replacement methods follows as:

	cap(self)  					   -- capitalizing first letter of word "self"
	rfind(self, substr)  		   -- find location of the last substring in a given string
	contains(self, substr)   	   -- check whether string "self" contains "substr" or not
	startsWith(self, begin)  	   -- check whether string "self" starts with "begin" or not
	endsWith(self, tail)     	   -- check whether string "self" ends with "tail" or not
	replace(self, ori, new, n) 	   -- replace substring with new substring
	mapreplace (self, mapping, n)  -- multiple replacements by mapping from old substring to new one
	
sometimes you will concatenate list of strings together. On the contrary, you want to split a long string into multiple smaller ones by delimiters. The following is a group of methods with spliting functionality, and `split()` is a specific one.     
	
	split(self, delim, count, no_patterns)   	-- spliting a given string by a delimiter and returning a list of splited pieces   
	splitOut(self, delim, count, no_patterns)   -- unpack a list of splited pieces that returned by split()   
	splitBy(self, ...)    						-- spliting a given string by several delimiters   
	
where `self` is the splited string, `delim` for delimiter, `count` for times that the delimiter could be replaced and `no_patterns` can be choosed as `true|false|nil`, which could turn off regular expression in delimiter or not.


Sometimes, there are some blank spaces at head or tail of handled strings. For security issue, we should clean them firstly before processing. `ltrim(self)` trims out the blank space at the head of string `self`, `rtrim(self)` trims out blank space at the tail of string `self` and `trim(self)` trims out blank space at both sides of string `self`.


UTF8 encoding and decoding 
every string generated by lua will be encoded in UTF8??? NO, Lua just can handle UTF8 bytes directly.


###Input/Output     
`loadFile(from_dir, name)` loading files into memory and the basic unit is file, while `loadLines(source, firstline, lastline)` loading a source file that indexed from `firstline` to `lastline` and prefixing line number for each line. And now the basic unit is line.


###Table ----*to be reused in terms of implementation rather than interface*

To add a length parameter of table, we just wrap simple methods of lua-table `insert()`, `remove()` and modify the constructor `{}`, `t["abc"] = "def"` and `t["abc"] = nil` operations with maintenance of the lenght parameter. This extra parameter can accelerate performance of many operations, such as, `equal()`, `merge()`, `difference()`, etc. By the way, Lua-5.2 add [__len metamethod](http://www.lua.org/work/doc/manual.html#3.4.6) for lua-table. For now, we can design a good API firstly. The implementation could be improved later on.

`isEmpty()` returns true for no element in lua-table, while false for at least one element. `update(source, keys)` updates itself by all key-value pairs of `source` table if `keys` is `nil`. Otherwise, only keys in a `keys`-list are updated.
 

**value vs. reference**     
`equal(another)` now is implemented only in one-layer. Therefore, for a plain table, it is equal by value; for table with nested structures, it is equal by reference for such fields. Be careful that value and reference are mixed in the sense of semantics. `copy()` returns a new copy of table, but still sharing nested tables with the orginal version because of one-layer copying. For nesting structure, nested tables are shared by the original and copy of tables. `deepCopy()` makes an exact value-copy of one table, and there are no references shared any more.


`takeAparts()` splits a lua-table into two parts, list-part and dict-part. The underlying reason of this spliting is that lua-table is implemented as a combination of a C-array and a hash-table internally (**C-array---->list and hash table ---->dict.**). On one hand, it is much better to be used as a combination, like implementation of queue/stack. On the other hand, using them separately could be also preferred, like keeping the inserted order (list-type) and much faster inserting/deleting operations (dict-type or set). Therefore, depending on your user case, you should choose to use a proper type, that is, set(ordered or unordered), dict(ordered or unordered), list or queue. 


The next question you may ask follows as. "how does lua store the data of lua-table?" IF table is a sequence or sequence part of lua-table, that is, the set of its positive numeric keys is equal to {1..n} for some integer n, it will store this piece into a C-array internally. We call this part as list-type because of its stroage. For all other cases, it is treated as dict-type. For example, any key that is not one of {1..n} will be stored in hash table. Also, if a key with value m (m>n), a hole case, will be also treated as dict type.  For example,      
	
	t = {1,2,3,4, one = 1, two = 2, three = 3, 5}
	t[0] = 0
	t[-1] = -1
	t[10] = 10
	
where 1, 2, 3, 4 and 5 are stored as array/list, and others including 0->0, -1->-1 and 10->10 are stored as dict/hash table. 

One more thing that you should keep in mind is the spliting done here only in one layer. The list/dict type that you get can also contains nesting lua-table internally. SOMETIMES one-layer is enough??? I think we should try to use customized APIs (**List, Dict, Set, Queue**) as much as possible rather than lua-table directly. 


## Data structure 
###List
List is the prototype of all List instances. It inherits the extended version of lua-table. Actually, list here contains two parts of APIs, array-part and queue/stack-part. It may be splited further in the future. In the following we will introduce common-part, array-part and queue-part one by one. 

For initialization, you can create an empty list by `List()` or a list holding several elements by `List(tbl)`. For later one, constructor only selects list-type of elements of table `tbl` to fill in a list. For example, 
	
	local lista = List()
	local listb = List {1,2,3,4,5,6}
	
`range(start, finish)` is a class method, where `start` parameter is optional, and its default value is 1. Both sides are inclusive and a sequence of integers from `start` to `finish` are generated. All others are object methods that operated on a specific object. `List:len()`
returns the size/length of a list. From the perspective of performance, storing and maintaining a length parameter may be a better choice. `List:isEmpty()` checks whether a list is empty or not and `List:clear()` can clear all elements of list. The magic method `__eq()` tests whether two lists are equal or not, and can be written as `l2 == l1`. For this magic method, there is the same issue of **value or reference** to be taken care of. That is all for the common part of APIs.


For the array part of APIs, let's explain the basic operations firstly, that is, insert/delete/read/update.
	
	List:splice(idx, list)   -- inserting another "list" at the location "idx" of List "self", and just list expansion by another one
	List:extend(another) 	 -- list concatenation can be written as lnew = l1 + l2, and a special case of List:splice(idx, list) appending at the tail of "self" list. 
	List:iremove (i)  		 -- deleting by index
	List:remove(x) 			 -- deleting all elements that have the value "x", which is a special case of List:remove(x, numOfDeletion)
	List:chop(i1,i2)    	 -- deleting by indexing interval
	
For `List:remove(x, numOfDeletion)`, if `numOfDeletion` is negative integer, it will start counting from the last one in reversed order. As for read and update operations, `List:find(val, idx)` find the first element with `value = val` by starting from index `idx`. `List:slice(start, stop, is_rev)` selects a piece of list elements with index from `start` to `stop`, where `start`, `stop` maybe nil, negative integer, or other values, and the default value of `start` is 1, `#list` for `stop`. If `is_rev` is "rev", the returned list is in reversed order.  `List:sliceAssign(i1, i2, seq)` is an assignment operation in the style of slicing. In addition to basic operations, there are some helper methods for each list instance:
	
	List:contains(x)  -- check whether a list instance has the element "x"
	List:count(x)  	  -- counting the times that element "x" repeats in the list "self"
	List:join(sep)    -- simpe wrapper of table.concat() method 
	List:sort(cmp)    -- sort a list w.r.t. an order function "cmp", and it is a simple wrapper of table.sort(orderFunction)
	List:reverse()    -- reversing the order of list elements 
	
	
Now we arrive at the queue-part of list section, which will be implemented as a lua-table later. The standard API follows as:
	
	List:append(val)   -- appending an extra element at the tail of list
	List:prepend(val)  -- appending an extra element at the head of list
	List:push(val) 	   -- push a new element into list from the right-hand side
	List:pop() 		   -- pop and remove a element from list from the right-hand side
	
Maybe some methods, like pushLeft(), popLeft(), elementHead() and elementTail(), will be added later for better use. One more thing that should be kept in mind is the element `x/val` is a basic value or reference.


###Dict      
Dict is the prototype of all Dict instances. It inherits the extended version of lua-table. Therefore Dict instance can use table API, especially dict-type of them. Like List prototype, the initialization of a Dict object/instance follows as:     
	
	dicta = {}
	dictb = {one = 1, two = 2, three = 3, four = 4}
	
>>Remarks: the relationship between Dict and table shoud be redefined as implementing the Dict API by extensive library of lua table and removing this inheritance relationship.

For the basic operations of single key-value pair, we had better to use table API, that is, 
	
	tt["abc"] = "cde" 		-- treated as insert or update
	tt["singled"] = nil		-- delete operation
	tt["double"]  			-- read operation
	
To achieve better performance, the key of key-value pair should be chosen as **lua string**. If key in key-value pair is string, the pair is guaranteed to be stored inside hash table internally. For Dict-specific methods,
	
	Dict:keys()    -- return all the keys of Dict instance **without** keeping any order because of feature of hash table.
	Dict:values()  -- return all the values of Dict instance, still without keeping any order
	Dict:hasKey()  -- check whether a specific key exist in a Dict instance or not
	Dict:size()    -- return the length of Dict instance, counting once again when called
	

###Set     
Set is the prototype of all Set instances. It inherits the prototype of Dict. Therefore Set instance can use table and Dict APIs, especially dict-style of them. Like List prototype, the initialization of a Set object/instance follows as: 
	
	seta = {} 
	setb = {"one", "two", "2", "three", "four"}
	
As for basic operations, same issue of keys with string type should be taken care. When encounting positive integers as elements, you should first explicitly convert them into string and then do necessary operations. This requirement results from the consideration of performance. We would like to store all elements into hash table internally. The same argument also holds for Dict type. For example, 
	
	seta:add("12") NOT seta:add(12)  		--->{"23" = true}
	setb:delete("2")  NOT setb:delete(2)    --->{"one", "two","three", "four"}

You can also use the table default APIs, like `seta["23"] = true` or `seta["abc"] = nil` to do the same things. Actually, methods `add()` and `delete()` are implemented in this way internally, that can be treated as raw APIs. `has(key)` tests whether a element `key` exist or not, `members()` just returns all of elements in random order. Finally, five logical operations of set follows as:
	
	Set:union(another)					-- standard union operation, can be written as A + B
	Set:intersection(another)			-- standard intersection operation, can be written as A * B
	Set:difference(another)				-- standard difference operation, can be written as A - B
	Set:symmetricDifference(another) 	-- standard symmetric difference operations, can be written as A ^ B
	Set:isSub(another)					-- check whether "self" is a sub-set of "another" or not, can be written as A < B
	

>>remarks: removing the inheritance relationship, Set should be at the same foot of Dict. isEmpty() constructed as a simple wrapper or re-implemented will be much better. Also reusing the length parameter of lua table as its size, the difficult task is to maintain the parameter. 


###Advanced ordered data structures    
lua-table with extension can be treated as building blocks for interface of [list, queue, set(sorted set), dict(sorted dict)].  The ordered and unordered verions of data structures form the following [tree] (http://loop.luaforge.net/library/collection/PriorityQueue.html), 
	
							  --------collection---------				  dict
							  |		      |				|					|
							list 	     set  		   queue			ordered dict			
							  			  |	  			 				  
							  		  sorted set  					  
	
Only sorted set and dict have not been discussed yet. For the sorted version, the key point is weight parameter, which can be key, value or extra weight parameter. As for the implementation, each table treated as one node can hold an extra weight parameter. In the constructor of ordered versions, client should choose/provide a weight parameter and related order function. Internally, all methods building the ordered data structure will refer to them when inserting/deletion/iteration/has/etc. Once implemented, sorted set (ordered by keys or weight parameter) and sorted dict (ordered by keys, values or weight parameter) are just simple wrappers or special cases of this implementation. In some sense, this implementation is at the same foot of lua-table.


## Object-Oriented Programming
Implementation of OOP mechanism has been constructed by lua table and metatable. `Object` is the most oldest parent for all of other class instance. It has several primitive fields [**__tag** and **__parent**] and methods [**new(), extend(), clone, equal(), init()**] built-in. Usually the class that user defined has the following structure:
	
	__tag  				-- description of a class, usually its name
	__parent    		-- the name of its parent class, stored the inheritance relationship
	name/age/sex/  		-- other usual fields that customers can defined

`__parent` holds the name of its parent class. This OOP mechanism is single-rooted, which means all of classes must be traced back to the `Object` class. When the length of inheritance chain is too long, methods called like `new()` and `extend()` take long time because of jumping several times in the backtrace. Maybe this issue can be solved if they can be treated as moudle methods to be called directly.
	
	new()		 -- a constructor of any class
	extend()	 -- method that only parent class has
	clone() 	 -- only dond it in one-layer deep, be careful about shared-references
	equal()		 -- hashCode() is used to be equal().. equal by value or reference??
	init()		 -- process along the inheritance relationship
	
Magic methods, like `__add`, `__mod`, `__div` and `__mul`, are used frequently because of convenience. Actually, magic methods still has the same issue of built-in methods call when searching methods in metatable in the backward direction. Our implementation of magic methods adds a reference/index cache, which could accelarate method call processes. Once methods modified, the change can not propagate into the reference cache, it will lost a little of flexibility. In a word, it just trade off between performance and flexibility.

Ordinary methods, `doSomething()` method could be defined for your customized usage. Up to now, the inheritance and composition of OOP have been discussed. We just use an example to illustrate the above concepts. 
	
	local Comment = Object:extend{....}		-- inheritance relationship
		__name = 'Comment';  
		__desc = 'General comment definition.';  
	  
		-- constructor
		init = function (self, t)  
		    if not t then return self end     
	  
		    self.type = t.type  
		    self.author = t.author  
		    self.content = t.content  
		    self.posts = Post()		-- instance composition
		    self.timestamp = t.timestamp or os.time()  
	  
		    return self  
		end;  
	  
		doSomething = function (self)  
		    .....  
		end;
	}  

	local Post = Object:extend{....}		-- inheritance relationship
		__name = 'Post';  
		__desc = 'General post definition.';  
	  
		-- constructor
		init = function (self, t)  
		    if not t then return self end     
	  
		    self.title = t.title  
		    self.author = t.author  
		    self.content = t.content  
		    self.timestamp = t.timestamp or os.time()  
	  
		    return self  
		end;  
	  
		doSomething = function (self)  
		    .....  
		end;
	}  
 

After definition, we can use it as following:
	
	local post_obj = Post()			-- create an empty post
	local msg_obj = Message()  		-- create an empty message
	local msg_obj2 = Message {  	-- create a message with contents  
		    type = '100',  
		    author = 'daoge',  
		    content = 'hello',  
		    posts = post_obj,
		}  

	msg_obj2:doSomething();     -- calling method defined in class instance  


####polymorphism
Composition reuses implementation, and inheritance defined here reuses implementation and interface at the same time. Now we can also write code in the style of reusing interface only. 
	
	require "Cube"		-- getSurface() and getVolume() defined in a Cube class
	require "Sphere"	-- getSurface() and getVolume() also defined in a Sphere class

	shapes = {
			Cube{10,15,20},
			Cube{12,14,16},
			Sphere{10},
			Sphere{17},
			Cube{2,4,26}
	    }

	for i, target in ipairs(shapes) do
		    s_area = target:getSurface()
		    volume = target:getVolume()
		    print (s_area, volume)
	end
	
	
## Helper methods
###Http          
`escapeHTML(s)`    
simple HTML escape sequence

`encodeURL(url)`     
simple URL encoding  

`decodeURL(url)`    
Simplistic URL decoding that can handle "plus" and "space" encoding too. ??

`parseURL(url, sep)`     
parse parameters of URL and store key-value pairs into lua-table
