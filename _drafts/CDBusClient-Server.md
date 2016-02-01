---
layout: "post"
title: "CDBusClient/Server"
---

> Introducing GDBus client-server model, CDBusClient-Server wrapper for inheriting purpose
<!--excerpt-->

# GDBus Client Init Workflow

1. `g_type_init`
  
2. `g_bus_get_sync`
  * there are two *bus_type*s available:
    * *G_BUS_TYPE_SYSTEM*: the system-wide message bus connection
    * *G_BUS_TYPE_SESSION*: the login session message bus connection
    currently only use *G_BUS_TYPE_SYSTEM*
  * return a connection to dbus (system-wide/loging session)

3. `create mainloop`
4. `create proxy`
5. `connect signal`

# GDBusClient wrapper

before `create proxy`, register two callbacks to wait 
