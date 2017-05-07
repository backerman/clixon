# Clixon CHANGELOG

## 3.3.0

May 2017	
	
- Datastore text module is now default.

- Refined netconf "none" semantics in tests and text datastore

- Moved apps/dbctrl to datastore/

- Added connect/disconnect/getopt/setopt and handle to xmldb API

- Added datastore 'text'

- Configure (autoconf) changes
  Removed libcurl dependency
  Disable restconf (and fastcgi) with configure --disable-restconf
  Disable keyvalue datastore (and qdbm) with configure --disable-keyvalue

- Created xmldb plugin api
  Moved qdbm, chunk and  xmldb to datastore keyvalue directories
  Removed all other clixon dependency on chunk code
	
- cli_copy_config added as generic cli command
- cli_show_config added as generic cli command
  Replace all show_confv*() and show_conf*() with cli_show_config()
  Example: replace:
     show_confv_as_json("candidate","/sender");
  with:
     cli_show_config("candidate","json","/sender");
- Alternative yang spec option -y added to all applications
- Many clicon special string functions have been removed
- The netconf support has been extended with lock/unlock
- clicon_rpc_call() has been removed and should be replaced by extending the
  internal netconf protocol. 
  See downcall() function in example/routing_cli.c and 
  routing_downcall() in example/routing_backend.c
- Replace clicon_rpc_xmlput with clicon_rpc_edit_config
- Removed xmldb daemon. All xmldb acceses is made backend daemon. 
  No direct accesses by clients to xmldb API.
  Instead use the rpc calls in clixon_proto_client.[ch]
  In clients (eg cli/netconf) replace xmldb_get() in client code with 
  clicon_rpc_get_config().
  If you use the vector arguments of xmldb_get(), replace as follows:
    xmldb_get(h, db, api_path, &xt, &xvec, &xlen);
  with
    clicon_rpc_get_config(h, dbstr, api_path, &xt);
    xpath_vec(xt, api_path, &xvec, &xlen)

- clicon_rpc_change() is replaced with clicon_rpc_edit_config().
  Note modify argument 5:
     clicon_rpc_change(h, db, op, apipath, "value") 
  to:
     clicon_rpc_edit_config(h, db, op, apipath, "<config>value</config>") 

- xmdlb_put_xkey() and xmldb_put_tree() have been folded into xmldb_put()
  Replace xmldb_put_xkey with xmldb_put as follows:
     xmldb_put_xkey(h, "candidate", cbuf_get(cb), str, OP_REPLACE);
  with
     clicon_xml_parse(&xml, "<config>%s</config>", str);
     xmldb_put(h, "candidate", OP_REPLACE, cbuf_get(cb), xml);
     xml_free(xml);

- Change internal protocol from clicon_proto.h to netconf.
  This means that the internal protocol defined in clixon_proto.[ch] is removed

- Netconf startup configuration support. Set CLICON_USE_STARTUP_CONFIG to 1 to
  enable. Eg, if backend_main is started with -CIr startup will be copied to
  running.

- Added ".." as valid step in xpath

- Use restconf format for internal xmldb keys. Eg /a/b=3,4

- List keys with special characters RFC 3986 encoded.	

- Replaced cli expand functions with single to multiple args
  This change is _not_ backward compatible
  This effects all calls to expand_dbvar() or user-defined
  expand callbacks

- Replaced cli callback functions with single arg to multiple args
  This change is _not_ backward compatible.
  You are affected if you 
  (1) use system callbacks (i.e. in clixon_cli_api.h)
  (2) write your own cli callbacks

  If you use cli callbacks, you need to rewrite cli callbacks from eg:
     load("Comment") <filename:string>,load_config_file("filename replace");
  to:
     load("Comment") <filename:string>,load_config_file("filename", "replace");

  If you write your own, you need to change the callback signature from;
    int cli_callback(clicon_handle h, cvec *vars, cg_var *arg)
  to:
    int cli_callback(clicon_handle h, cvec *vars, cvec *argv)
  and rewrite the code to handle argv instead of arg.
  These are the system functions affected:
  cli_set, cli_merge, cli_del, cli_debug_backend, cli_set_mode, 
  cli_start_shell, cli_quit, cli_commit, cli_validate, compare_dbs, 
  load_config_file, save_config_file, delete_all, discard_changes, cli_notify,
  show_yang, show_conf_xpath

- Added --with-cligen and --with-qdbm configure options
- Added union type check for non-cli (eg xml) input 
- Empty yang type. Relaxed yang types for unions, eg two strings with different length.
	
Dec 2016: Dual license: both GPLv3 and APLv2
	
Feb 2016: Forked new clixon repository from clicon
