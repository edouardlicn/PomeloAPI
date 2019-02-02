#Application
Application prototype.
getBase
Application.getBase()
type: method
Get application base path
// cwd: /home/game/ pomelo start // app.getBase() -> /home/game
Source
Application.getBase = function() {
  return this.get(Constants.Reserved.BASE);
};

filter
Application.filter()
type: method
Params
filter - provide before and after filter method.
add a filter to before and after filter
Source
>Application.filter = function (filter) {
>  this.before(filter);
>  this.after(filter);
>};

before
Application.before()
type: method
Params
bf - before fileter, bf(msg, session, next)
Add before filter.
Source
Application.before = function (bf) {
  addFilter(this, Constants.KeyWords.BEFORE_FILTER, bf);
};

after
Application.after()
type: method
Params
af - after filter, `af(err, msg, session, resp, next)`
Add after filter.
Source
Application.after = function (af) {
  addFilter(this, Constants.KeyWords.AFTER_FILTER, af);
};

globalFilter
Application.globalFilter()
type: method
Params
filter - provide before and after filter method.
add a global filter to before and after global filter
Source
Application.globalFilter = function (filter) {
  this.globalBefore(filter);
  this.globalAfter(filter);
};

globalBefore
Application.globalBefore()
type: method
Params
bf - before fileter, bf(msg, session, next)
Add global before filter.
Source
Application.globalBefore = function (bf) {
  addFilter(this, Constants.KeyWords.GLOBAL_BEFORE_FILTER, bf);
};

globalAfter
Application.globalAfter()
type: method
Params
af - after filter, `af(err, msg, session, resp, next)`
Add global after filter.
Source
Application.globalAfter = function (af) {
  addFilter(this, Constants.KeyWords.GLOBAL_AFTER_FILTER, af);
};

rpcBefore
Application.rpcBefore()
type: method
Params
bf - before fileter, bf(serverId, msg, opts, next)
Add rpc before filter.
Source
Application.rpcBefore = function(bf) {
  addFilter(this, Constants.KeyWords.RPC_BEFORE_FILTER, bf);
};

rpcAfter
Application.rpcAfter()
type: method
Params
af - after filter, `af(serverId, msg, opts, next)`
Add rpc after filter.
Source
Application.rpcAfter = function(af) {
  addFilter(this, Constants.KeyWords.RPC_AFTER_FILTER, af);
};

rpcFilter
Application.rpcFilter()
type: method
Params
filter - provide before and after filter method.
add a rpc filter to before and after rpc filter
Source
Application.rpcFilter = function(filter) {
  this.rpcBefore(filter);
  this.rpcAfter(filter);
};

load
Application.load()
type: method
Params
name - (optional) name of the component
component - component instance or factory function of the component
opts - (optional) construct parameters for the factory function
Load component
Source
Application.load = function(name, component, opts) {
  if(typeof name !== 'string') {
    opts = component;
    component = name;
    name = null;
    if(typeof component.name === 'string') {
      name = component.name;
    }
  }

  if(typeof component === 'function') {
    component = component(this, opts);
  }

  if(!name && typeof component.name === 'string') {
    name = component.name;
  }

  if(name && this.components[name]) {
    // ignore duplicat component
    logger.warn('ignore duplicate component: %j', name);
    return;
  }

  this.loaded.push(component);
  if(name) {
    // components with a name would get by name throught app.components later.
    this.components[name] = component;
  }

  return this;
};

loadConfig
Application.loadConfig()
type: method
Params
key - environment key
val - environment value
Load Configure json file to settings.
Source
Application.loadConfig = function (key, val) {
  var env = this.get(Constants.Reserved.ENV);
  val = require(val);
  if (val[env]) {
    val = val[env];
  }
  this.set(key, val);
};

route
Application.route()
type: method
Params
serverType - server type string
routeFunc - route function. routeFunc(session, msg, app, cb)
Set the route function for the specified server type.
Examples:
app.route('area', routeFunc);
var routeFunc = function(session, msg, app, cb) { // all request to area would be route to the first area server var areas = app.getServersByType('area'); cb(null, areas[0].id); };
Source
Application.route = function(serverType, routeFunc) {
  var routes = this.get(Constants.KeyWords.ROUTE);
  if(!routes) {
    routes = {};
    this.set(Constants.KeyWords.ROUTE, routes);
  }
  routes[serverType] = routeFunc;
  return this;
};

beforeStopHook
Application.beforeStopHook()
type: method
Params
fun - before close function
Set before stop function. It would perform before servers stop.
Source
Application.beforeStopHook = function(fun) {
  logger.warn('this method was deprecated in pomelo 0.8');
  if(!!fun && typeof fun === 'function') {
    this.set(Constants.KeyWords.BEFORE_STOP_HOOK, fun);
  }
};

start
Application.start()
type: method
Params
cb - callback function
Start application. It would load the default components and start all the loaded components.
Source
Application.start = function(cb) {
  this.startTime = Date.now();
  if(this.state > STATE_INITED) {
    utils.invokeCallback(cb, new Error('application has already start.'));
    return;
  }
  appUtil.loadDefaultComponents(this);
  var self = this;
  var startUp = function() {
    appUtil.optComponents(self.loaded, Constants.Reserved.START, function(err) {
      self.state = STATE_START;
      if(err) {
        utils.invokeCallback(cb, err);
      } else {
        logger.info('%j enter after start...', self.getServerId());
        self.afterStart(cb);
      }
    });
  };
  var beforeFun = this.lifecycleCbs[Constants.LIFECYCLE.BEFORE_STARTUP];
  if(!!beforeFun) {
    beforeFun.call(null, this, startUp);
  } else {
    startUp();
  }
};

set
Application.set()
type: method
Params
setting - the setting of application
val - the setting's value
attach - whether attach the settings to application
Assign setting to val, or return setting's value.
Example:
app.set('key1', 'value1'); app.get('key1'); // 'value1' app.key1; // undefined
app.set('key2', 'value2', true); app.get('key2'); // 'value2' app.key2; // 'value2'
Source
Application.set = function (setting, val, attach) {
  if (arguments.length === 1) {
    return this.settings[setting];
  }
  this.settings[setting] = val;
  if(attach) {
    this[setting] = val;
  }
  return this;
};

get
Application.get()
type: method
Params
setting - application setting
Get property from setting
Source
Application.get = function (setting) {
  return this.settings[setting];
};

enabled
Application.enabled()
type: method
Params
setting - application setting
Check if setting is enabled.
Source
Application.enabled = function (setting) {
  return !!this.get(setting);
};

disabled
Application.disabled()
type: method
Params
setting - application setting
Check if setting is disabled.
Source
Application.disabled = function (setting) {
  return !this.get(setting);
};

enable
Application.enable()
type: method
Params
setting - application setting
Enable setting.
Source
Application.enable = function (setting) {
  return this.set(setting, true);
};

disable
Application.disable()
type: method
Params
setting - application setting
Disable setting.
Source
Application.disable = function (setting) {
  return this.set(setting, false);
};

configure
Application.configure()
type: method
Params
env - application environment
fn - callback function
type - server type
Configure callback for the specified env and server type. When no env is specified that callback will be invoked for all environments and when no type is specified that callback will be invoked for all server types.
Examples:
app.configure(function(){ // executed for all envs and server types });
app.configure('development', function(){ // executed development env });
app.configure('development', 'connector', function(){ // executed for development env and connector server type });
Source
Application.configure = function (env, type, fn) {
  var args = [].slice.call(arguments);
  fn = args.pop();
  env = type = Constants.Reserved.ALL;

  if(args.length > 0) {
    env = args[0];
  }
  if(args.length > 1) {
    type = args[1];
  }

  if (env === Constants.Reserved.ALL || contains(this.settings.env, env)) {
    if (type === Constants.Reserved.ALL || contains(this.settings.serverType, type)) {
      fn.call(this);
    }
  }
  return this;
};

registerAdmin
Application.registerAdmin()
type: method
Params
module - (optional) module id or provoided by module.moduleId
module - module object or factory function for module
opts - construct parameter for module
Register admin modules. Admin modules is the extends point of the monitor system.
Source
Application.registerAdmin = function(moduleId, module, opts) {
  var modules = this.get(Constants.KeyWords.MODULE);
  if(!modules) {
    modules = {};
    this.set(Constants.KeyWords.MODULE, modules);
  }

  if(typeof moduleId !== 'string') {
    opts = module;
    module = moduleId;
    if(module) {
      moduleId = module.moduleId;
    }
  }

  if(!moduleId){
    return;
  }

  modules[moduleId] = {
    moduleId: moduleId,
    module: module,
    opts: opts
  };
};

use
Application.use()
type: method
Params
plugin - plugin instance
opts - (optional) construct parameters for the factory function
Use plugin.
Source
Application.use = function(plugin, opts) {
  if(!plugin.components) {
    logger.error('invalid components, no components exist');
    return;
  }
  
  var self = this;
  var dir = path.dirname(plugin.components);

  if(!fs.existsSync(plugin.components)) {
    logger.error('fail to find components, find path: %s', plugin.components);
    return;
  }

  fs.readdirSync(plugin.components).forEach(function (filename) {
    if (!/\.js$/.test(filename)) {
      return;
    }
    var name = path.basename(filename, '.js');
    var param = opts[name] || {};
    var absolutePath = path.join(dir, Constants.Dir.COMPONENT, filename);
    if(!fs.existsSync(absolutePath)) {
      logger.error('component %s not exist at %s', name, absolutePath);
    } else {
      self.load(require(absolutePath), param);
    }
  });

  // load events
  if(!plugin.events) {
    return;
  } else {
    if(!fs.existsSync(plugin.events)) {
      logger.error('fail to find events, find path: %s', plugin.events);
      return;
    }

    fs.readdirSync(plugin.events).forEach(function (filename) {
      if (!/\.js$/.test(filename)) {
        return;
      }
      var absolutePath = path.join(dir, Constants.Dir.EVENT, filename);
      if(!fs.existsSync(absolutePath)) {
        logger.error('events %s not exist at %s', filename, absolutePath);
      } else {
        bindEvents(require(absolutePath), self);
      }
    });
  }
};

transaction
Application.transaction()
type: method
Params
name - transaction name
conditions - functions which are called before transaction
handlers - functions which are called during transaction
retry - retry times to execute handlers if conditions are successfully executed
Application transaction. Transcation includes conditions and handlers, if conditions are satisfied, handlers would be executed. And you can set retry times to execute handlers. The transaction log is in file logs/transaction.log.
Source
Application.transaction = function(name, conditions, handlers, retry) {
  appManager.transaction(name, conditions, handlers, retry);
};

getMaster
Application.getMaster()
type: method
Get master server info.
Source
Application.getMaster = function() {
  return this.master;
};

getCurServer
Application.getCurServer()
type: method
Get current server info.
Source
Application.getCurServer = function() {
  return this.curServer;
};

getServerId
Application.getServerId()
type: method
Get current server id.
Source
Application.getServerId = function() {
  return this.serverId;
};

getServerType
Application.getServerType()
type: method
Get current server type.
Source
Application.getServerType = function() {
  return this.serverType;
};

getServers
Application.getServers()
type: method
Get all the current server infos.
Source
Application.getServers = function() {
  return this.servers;
};

getServersFromConfig
Application.getServersFromConfig()
type: method
Get all server infos from servers.json.
Source
Application.getServersFromConfig = function() {
  return this.get(Constants.KeyWords.SERVER_MAP);
};

getServerTypes
Application.getServerTypes()
type: method
Get all the server type.
Source
Application.getServerTypes = function() {
  return this.serverTypes;
};

getServerById
Application.getServerById()
type: method
Params
serverId - server id
Get server info by server id from current server cluster.
Source
Application.getServerById = function(serverId) {
  return this.servers[serverId];
};

getServerFromConfig
Application.getServerFromConfig()
type: method
Params
serverId - server id
Get server info by server id from servers.json.
Source
Application.getServerFromConfig = function(serverId) {
  return this.get(Constants.keyWords.SERVER_MAP)[serverId];
};

getServersByType
Application.getServersByType()
type: method
Params
serverType - server type
Get server infos by server type.
Source
Application.getServersByType = function(serverType) {
  return this.serverTypeMaps[serverType];
};

isFrontend
Application.isFrontend()
type: method
Params
server - server info. it would check current server
Check the server whether is a frontend server
Source
Application.isFrontend = function(server) {
  server = server || this.getCurServer();
  return !!server && server.frontend === 'true';
};

isBackend
Application.isBackend()
type: method
Params
server - server info. it would check current server
Check the server whether is a backend server
Source
Application.isBackend = function(server) {
  server = server || this.getCurServer();
  return !!server && !server.frontend;
};

isMaster
Application.isMaster()
type: method
Check whether current server is a master server
Source
Application.isMaster = function() {
  return this.serverType === Constants.Reserved.MASTER;
};

addServers
Application.addServers()
type: method
Params
servers - new server info list
Add new server info to current application in runtime.
Source
Application.addServers = function(servers) {
  if(!servers || !servers.length) {
    return;
  }

  var item, slist;
  for(var i=0, l=servers.length; i < l; i++) {
    item = servers[i];
    // update global server map
    this.servers[item.id] = item;

    // update global server type map
    slist = this.serverTypeMaps[item.serverType];
    if(!slist) {
      this.serverTypeMaps[item.serverType] = slist = [];
    }
    replaceServer(slist, item);

    // update global server type list
    if(this.serverTypes.indexOf(item.serverType) < 0) {
      this.serverTypes.push(item.serverType);
    }
  }
  this.event.emit(events.ADD_SERVERS, servers);
};

removeServers
Application.removeServers()
type: method
Params
ids - server id list
Remove server info from current application at runtime.
Source
Application.removeServers = function(ids) {
  if(!ids || !ids.length) {
    return;
  }

  var id, item, slist;
  for(var i=0, l=ids.length; i < l; i++) {
    id = ids[i];
    item = this.servers[id];
    if(!item) {
      continue;
    }
    // clean global server map
    delete this.servers[id];

    // clean global server type map
    slist = this.serverTypeMaps[item.serverType];
    removeServer(slist, id);
    // TODO: should remove the server type if the slist is empty?
  }
  this.event.emit(events.REMOVE_SERVERS, ids);
};

replaceServers
Application.replaceServers()
type: method
Params
server - id map
Replace server info from current application at runtime.
Source
Application.replaceServers = function(servers) {
  if(!servers){
    return;
  }

  this.servers = servers;
  this.serverTypeMaps = {};
  this.serverTypes = [];
  var serverArray = [];
  for(var id in servers){
    var server = servers[id];
    var serverType = server[Constants.Reserved.SERVER_TYPE];
    var slist = this.serverTypeMaps[serverType];
    if(!slist) {
      this.serverTypeMaps[serverType] = slist = [];
    }
    this.serverTypeMaps[serverType].push(server);
    // update global server type list
    if(this.serverTypes.indexOf(serverType) < 0) {
      this.serverTypes.push(serverType);
    }
    serverArray.push(server);
  }
  this.event.emit(events.REPLACE_SERVERS, serverArray);
};

addCrons
Application.addCrons()
type: method
Params
crons - new crons would be added in application
Add crons from current application at runtime.
Source
Application.addCrons = function(crons) {
  if(!crons || !crons.length) {
    logger.warn('crons is not defined.');
    return;
  }
  this.event.emit(events.ADD_CRONS, crons);
};

removeCrons
Application.removeCrons()
type: method
Params
crons - old crons would be removed in application
Remove crons from current application at runtime.
Source
Application.removeCrons = function(crons) {
  if(!crons || !crons.length) {
    logger.warn('ids is not defined.');
    return;
  }
  this.event.emit(events.REMOVE_CRONS, crons);
};

var replaceServer = function(slist, serverInfo) {
  for(var i=0, l=slist.length; i < l; i++) {
    if(slist[i].id === serverInfo.id) {
      slist[i] = serverInfo;
      return;
    }
  }
  slist.push(serverInfo);
};

var removeServer = function(slist, id) {
  if(!slist || !slist.length) {
    return;
  }

  for(var i=0, l=slist.length; i < l; i++) {
    if(slist[i].id === id) {
      slist.splice(i, 1);
      return;
    }
  }
};

var contains = function(str, settings) {
  if(!settings) {
    return false;
  }

  var ts = settings.split("|");
  for(var i=0, l=ts.length; i < l; i++) {
    if(str === ts[i]) {
      return true;
    }
  }
  return false;
};

var bindEvents = function(Event, app) {
  var emethods = new Event(app);
  for(var m in emethods) {
    if(typeof emethods[m] === 'function') {
      app.event.on(m, emethods[m].bind(emethods));
    }
  }
};

var addFilter = function(app, type, filter) {
 var filters = app.get(type);
  if(!filters) {
    filters = [];
    app.set(type, filters);
  }
  filters.push(filter);
};


#BackendSessionService

Service that maintains backend sessions and the communiation with frontend servers.
BackendSessionService would be created in each server process and maintains backend sessions for current process and communicates with the relative frontend servers.
BackendSessionService instance could be accessed by app.get('backendSessionService') or app.backendSessionService.
get
BackendSessionService.prototype.get()
type: method
Params
frontendId - frontend server id that session attached
sid - session id
cb - callback function. args: cb(err, BackendSession)
Get backend session by frontend server id and session id.
Source
BackendSessionService.prototype.get = function(frontendId, sid, cb) {
  var namespace = 'sys';
  var service = 'sessionRemote';
  var method = 'getBackendSessionBySid';
  var args = [sid];
  rpcInvoke(this.app, frontendId, namespace, service, method,
            args, BackendSessionCB.bind(null, this, cb));
};

getByUid
BackendSessionService.prototype.getByUid()
type: method
Params
frontendId - frontend server id that session attached
uid - user id binded with the session
cb - callback function. args: cb(err, BackendSessions)
Get backend sessions by frontend server id and user id.
Source
BackendSessionService.prototype.getByUid = function(frontendId, uid, cb) {
  var namespace = 'sys';
  var service = 'sessionRemote';
  var method = 'getBackendSessionsByUid';
  var args = [uid];
  rpcInvoke(this.app, frontendId, namespace, service, method,
            args, BackendSessionCB.bind(null, this, cb));
};

kickBySid
BackendSessionService.prototype.kickBySid()
type: method
Params
frontendId - cooperating frontend server id
sid - session id
cb - callback function
Kick a session by session id.
Source
BackendSessionService.prototype.kickBySid = function(frontendId, sid, cb) {
  var namespace = 'sys';
  var service = 'sessionRemote';
  var method = 'kickBySid';
  var args = [sid];
  rpcInvoke(this.app, frontendId, namespace, service, method, args, cb);
};

kickByUid
BackendSessionService.prototype.kickByUid()
type: method
Params
frontendId - cooperating frontend server id
uid - user id
cb - callback function
Kick sessions by user id.
Source
BackendSessionService.prototype.kickByUid = function(frontendId, uid, cb) {
  var namespace = 'sys';
  var service = 'sessionRemote';
  var method = 'kickByUid';
  var args = [uid];
  rpcInvoke(this.app, frontendId, namespace, service, method, args, cb);
};

BackendSession
BackendSession is the proxy for the frontend internal session passed to handlers and it helps to keep the key/value pairs for the server locally. Internal session locates in frontend server and should not be accessed directly.
The mainly operation on backend session should be read and any changes happen in backend session is local and would be discarded in next request. You have to push the changes to the frontend manually if necessary. Any push would overwrite the last push of the same key silently and the changes would be saw in next request. And you have to make sure the transaction outside if you would push the session concurrently in different processes.
See the api below for more details.
bind
BackendSession.prototype.bind()
type: method
Params
uid - user id
cb - callback function
Bind current session with the user id. It would push the uid to frontend server and bind uid to the frontend internal session.
Source
BackendSession.prototype.bind = function(uid, cb) {
  var self = this;
  this.__sessionService__.bind(this.frontendId, this.id, uid, function(err) {
    if(!err) {
      self.uid = uid;
    }
    utils.invokeCallback(cb, err);
  });
};

unbind
BackendSession.prototype.unbind()
type: method
Params
uid - user id
cb - callback function
Unbind current session with the user id. It would push the uid to frontend server and unbind uid from the frontend internal session.
Source
BackendSession.prototype.unbind = function(uid, cb) {
  var self = this;
  this.__sessionService__.unbind(this.frontendId, this.id, uid, function(err) {
    if(!err) {
      self.uid = null;
    }
    utils.invokeCallback(cb, err);
  });
};

set
BackendSession.prototype.set()
type: method
Params
key - key
value - value
Set the key/value into backend session.
Source
BackendSession.prototype.set = function(key, value) {
  this.settings[key] = value;
};

get
BackendSession.prototype.get()
type: method
Params
key - key
Get the value from backend session by key.
Source
BackendSession.prototype.get = function(key) {
  return this.settings[key];
};

push
BackendSession.prototype.push()
type: method
Params
key - key
cb - callback function
Push the key/value in backend session to the front internal session.
Source
BackendSession.prototype.push = function(key, cb) {
  this.__sessionService__.push(this.frontendId, this.id, key, this.get(key), cb);
};

pushAll
BackendSession.prototype.pushAll()
type: method
Params
cb - callback function
Push all the key/values in backend session to the frontend internal session.
Source
BackendSession.prototype.pushAll = function(cb) {
  this.__sessionService__.pushAll(this.frontendId, this.id, this.settings, cb);
};

ChannelService
Create and maintain channels for server local.
ChannelService is created by channel component which is a default loaded component of pomelo and channel service would be accessed by app.get('channelService').
createChannel
ChannelService.prototype.createChannel()
type: method
Params
name - channel's name
Create channel with name.
Source
ChannelService.prototype.createChannel = function(name) {
  if(this.channels[name]) {
    return this.channels[name];
  }

  var c = new Channel(name, this);
  this.channels[name] = c;
  return c;
};

getChannel
ChannelService.prototype.getChannel()
type: method
Params
name - channel's name
create - if true, create channel
Get channel by name.
Source
ChannelService.prototype.getChannel = function(name, create) {
  var channel = this.channels[name];
  if(!channel && !!create) {
    channel = this.channels[name] = new Channel(name, this);
  }
  return channel;
};

destroyChannel
ChannelService.prototype.destroyChannel()
type: method
Params
name - channel name
Destroy channel by name.
Source
ChannelService.prototype.destroyChannel = function(name) {
  delete this.channels[name];
};

pushMessageByUids
ChannelService.prototype.pushMessageByUids()
type: method
Params
route - message route
msg - message that would be sent to client
uids - the receiver info list, [{uid: userId, sid: frontendServerId}]
opts - user-defined push options, optional
cb - cb(err)
Push message by uids. Group the uids by group. ignore any uid if sid not specified.
Source
ChannelService.prototype.pushMessageByUids = function(route, msg, uids, opts, cb) {
  if(typeof route !== 'string') {
    cb = opts;
    opts = uids;
    uids = msg;
    msg = route;
    route = msg.route;
  }

  if(!cb && typeof opts === 'function') {
    cb = opts;
    opts = {};
  }

  if(!uids || uids.length === 0) {
    utils.invokeCallback(cb, new Error('uids should not be empty'));
    return;
  }
  var groups = {}, record;
  for(var i=0, l=uids.length; i < l; i++) {
    record = uids[i];
    add(record.uid, record.sid, groups);
  }

  sendMessageByGroup(this, route, msg, groups, opts, cb);
};

broadcast
ChannelService.prototype.broadcast()
type: method
Params
stype - frontend server type string
route - route string
msg - message
opts - user-defined broadcast options, optional
cb - callback
Broadcast message to all the connected clients.
Source
ChannelService.prototype.broadcast = function(stype, route, msg, opts, cb) {
  var app = this.app;
  var namespace = 'sys';
  var service = 'channelRemote';
  var method = 'broadcast';
  var servers = app.getServersByType(stype);

  if(!servers || servers.length === 0) {
    // server list is empty
    utils.invokeCallback(cb);
    return;
  }

  var count = servers.length;
  var successFlag = false;

  var latch = countDownLatch.createCountDownLatch(count, function() {
    if(!successFlag) {
      utils.invokeCallback(cb, new Error('broadcast fails'));
      return;
    }
    utils.invokeCallback(cb, null);
  });

  var genCB = function() {
    return function(err) {
      if(err) {
        logger.error('[broadcast] fail to push message, err:' + err.stack);
        latch.done();
        return;
      }
      successFlag = true;
      latch.done();
    };
  };

  opts = {type: 'broadcast', userOptions: opts || {}};

  // for compatiblity 
  opts.isBroadcast = true;
  if(opts.userOptions) {
    opts.binded = opts.userOptions.binded;
    opts.filterParam = opts.userOptions.filterParam;
  }

  for(var i=0, l=count; i < l; i++) {
    app.rpcInvoke(servers[i].id, {namespace: namespace, service: service,
      method: method, args: [route, msg, opts]}, genCB());
  }
};

Channel
Channel maintains the receiver collection for a subject. You can add users into a channel and then broadcast message to them by channel.
add
Channel.prototype.add()
type: method
Params
uid - user id
sid - frontend server id which user has connected to
Add user to channel.
Source
Channel.prototype.add = function(uid, sid) {
  if(this.state > ST_INITED) {
    return false;
  } else {
    var res = add(uid, sid, this.groups);
    if(res) {
      this.records[uid] = {sid: sid, uid: uid};
    }
    return res;
  }
};

leave
Channel.prototype.leave()
type: method
Params
uid - user id
sid - frontend server id which user has connected to.
Remove user from channel.
Source
Channel.prototype.leave = function(uid, sid) {
  if(!uid || !sid) {
    return false;
  }
  delete this.records[uid];
  var res = deleteFrom(uid, sid, this.groups[sid]);
  if(this.groups[sid] && this.groups[sid].length === 0) {
    delete this.groups[sid];
  }
  return res;
};

getMembers
Channel.prototype.getMembers()
type: method
Get channel members.
Notice: Heavy operation.
Source
Channel.prototype.getMembers = function() {
  var res = [], groups = this.groups;
  var group, i, l;
  for(var sid in groups) {
    group = groups[sid];
    for(i=0, l=group.length; i < l; i++) {
      res.push(group[i]);
    }
  }
  return res;
};

getMember
Channel.prototype.getMember()
type: method
Params
uid - user id
Get Member info.
Source
Channel.prototype.getMember = function(uid) {
  return this.records[uid];
};

destroy
Channel.prototype.destroy()
type: method
Destroy channel.
Source
Channel.prototype.destroy = function() {
  this.state = ST_DESTROYED;
  this.__channelService__.destroyChannel(this.name);
};

pushMessage
Channel.prototype.pushMessage()
type: method
Params
route - message route
msg - message that would be sent to client
opts - user-defined push options, optional
cb - callback function
Push message to all the members in the channel
Source
Channel.prototype.pushMessage = function(route, msg, opts, cb) {
  if(this.state !== ST_INITED) {
    utils.invokeCallback(new Error('channel is not running now'));
    return;
  }

  if(typeof route !== 'string') {
    cb = opts;
    opts = msg;
    msg = route;
    route = msg.route;
  }

  if(!cb && typeof opts === 'function') {
    cb = opts;
    opts = {};
  }

  sendMessageByGroup(this.__channelService__, route, msg, this.groups, opts, cb);
};

SessionService
Session service maintains the internal session for each client connection.
Session service is created by session component and is only available in frontend servers. You can access the service by app.get('sessionService') or app.sessionService in frontend servers.
kick
SessionService.prototype.kick()
type: method
Params
uid - user id asscociated with the session
cb - callback function
Kick all the session offline under the user id.
Source
SessionService.prototype.kick = function(uid, reason, cb) {
  // compatible for old kick(uid, cb);
  if(typeof reason === 'function') {
    cb = reason;
    reason = 'kick';
  }
  var sessions = this.getByUid(uid);

  if(sessions) {
    // notify client
    for(var i=0, l=sessions.length; i < l; i++) {
      sessions[i].closed(reason);
    }
    process.nextTick(function() {
      utils.invokeCallback(cb);
    });
  } else {
    process.nextTick(function() {
      utils.invokeCallback(cb);
    });
  }
};

kickBySessionId
SessionService.prototype.kickBySessionId()
type: method
Params
sid - session id
cb - callback function
Kick a user offline by session id.
Source
SessionService.prototype.kickBySessionId = function(sid, cb) {
  var session = this.get(sid);

  if(session) {
    // notify client
    session.closed('kick');
    process.nextTick(function() {
      utils.invokeCallback(cb);
    });
  } else {
    process.nextTick(function() {
      utils.invokeCallback(cb);
    });
  }
};

Pomelo
Expose createApplication().
createApp
Pomelo.createApp()
type: method
Create an pomelo application.
Source
Pomelo.createApp = function (opts) {
  var app = application;
  app.init(opts);
  self.app = app;
  return app;
};