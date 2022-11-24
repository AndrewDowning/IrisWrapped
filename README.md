# IrisWrapped
A Pythonic wrapper for Intersystems IrisNative API

Provides the sort of interface semantics that most Python programmers would expect.
For example:
1. Resource Acquisition Is Instantiation:  
    with Iris(ip=args.ip, port=args.port, namespace=args.namespace,
              username=args.username, password=args.password) as iris:
        ... do stuff knowing that on exit, the iris connection will be gone ...
    with iris.transaction() as tran:
        ... do database stuff knowing that on exit, the transaction will either
            be committed or aborted, but not dangling ...
        Note: Also multi-level transactions.

2. Exceptions:
    try:
        with Iris(ip=args.ip, port=args.port, namespace=args.namespace,
                  username=args.username, password=args.password) as iris:
            ... blah blah ...
    except IrisConnectionException as e:        # Deal with failure to connect.
        print(repr(e))

3. An IrisGlobal type that represents actual Iris Globals:
        with Iris(ip=args.ip, port=args.port, namespace=args.namespace,
                  username=args.username, password=args.password) as iris:
            iris.MyGlob = 42           # Creates an Iris global ^MyGlob and assigns the value 42.
            iris.MyGlob[(1,2,3)] = 5   # Directly applies tuples as Iris global subscripts
            
            # ... or with local Python variables:
            my_glob = IrisGlobal(iris, "MyGlob")  # Note you can't have _'s in Iris global names.
            my_glob[None] = 1
            my_glob[1, 2] = 3

            my_glob.name()                              # -> "^MyGlob"
            my_glob[None] = 42                          # set ^MyGlob = 42
            x = my_glob[None]                           # set x = ^MyGlob           except x is in Python
            my_glob[1,2,3] = 42                         # set ^MyGlob(1,2,3) = 42
            tup = (1,2,3)
            my_glob[*tup] = 42                          # set ^MyGlob(1,2,3) = 42
            x = my_glob[1,2,3]                          # set x = ^MyGlob(1,2,3)    except x is in Python
            x = my_glob(1,2,3)                          # set x = ^MyGlob(1,2,3)    except x is in Python
            del my_glob[1,2,3]                          # kill ^MyGlob(1,2,3)       Python style
            my_glob.kill((1,2,3))                       # kill ^MyGlob(1,2,3)       Closer to Object Script style
            my_glob.increment((1,2), 1)                 # $Increment(^MyGlob(1,2))
            my_glob.lock((1,2,3))                       # lock ^MyGlob(1,2,3)
            my_glob.unlock((1,2,3))                     # unlock ^MyGlob(1,2,3)
            my_glob.has_child((1,2,3))                  # $Data(^MyGlob(1,2,3)) = 10 or 11
            my_glob.has_value((1,2,3))                  # $Data(^MyGlob(1,2,3)) = 1  or 11
 
4. Iteration:
    for k,v in my_glob.iteritems()                          # $Order iterate key,value pairs at root of ^MyGlob
    for k,v in my_glob.iteritems(reverse=True)              # $Order iterate in reverse
    for k,v in my_glob.iteritems(key=(1,2), start_from=1)   # $Order iterate in subscripts below (1,2), starting from (1,2,1).
    for k   in my_glob.iterkeys((1,2))                      # $Order iterate key             below subroot of [1,2]
    for v   in my_glob.itervalues((1,))                     # $Order iterate value           at root of ^MyGlob
    for k,v in my_glob.iter_all(key=None)                   # Depth first iterate entire global
    

