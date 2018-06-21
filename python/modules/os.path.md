
# os.path的用法

```
>>> os.path.abspath(path)

'/usr/local/src/doc'

```
```
>>> os.path.basename('/root/doc/module')

'module'
```

os.path.commonpath(path)

os.path.commonprefix(list)

os.path.dirname(path)

os.path.exists(path)

os.path.lexists(path)

os.path.expanduser(path)

os.path.expandvars(path)

os.path.getatime(path)

os.path.getmtime(path)

os.path.getctime(path)

os.path.getsize(path)

os.path.isabs(path)

os.path.isfile(path)

os.path.isdir(path)

os.path.islink(path)

os.path.ismount(path)

os.path.join(path, *paths)

os.path.normcase(path)

os.path.normpath(path)

os.path.realpath(path)

os.path.relpath(path, start=os.curdir)

os.path.samefile(path1, path2)

os.path.sameopenfile(fp1, fp2)

os.path.samestat(stat1, stat2)

os.path.split(path)

os.path.splitdrive(path)

os.path.splitext(path)

os.path.splitunc(path)

os.path.supports_unicode_filenames
