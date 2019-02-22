 # samba:smbldap 

## roberto@edt asix m06 curs 2018-2019
 
### descripcion:

repositorio que contiene un servidor samba con backend ldap, de tal manera que hace un populate y cargara las dades de los usuarios desde ldap

#### configuracion 
* configurar los documentos nslcd y nsswitch para poder tener conectividad con ldap 

* configuracion del smb.conf en el cual añadiremos que nuestra base de datos vendra determinada por el ldap 

* configurar el servidor samba que ahora incorpora smbldap-tools y poder hacer el backend con ldap, ficheros principales smbldap.conf smbldap_bind.conf y el intall.sh donde haremos instalaciones necesarias para el funcionamiento correcto del montaje de los homes del los usuarios.

  	+ unix users samba necesitan la existencia de usuarios unix por tanto hay que disponer de un usuario unix locales o via ldap, con lo cual hay que configurar nscd y nslcd para la conectividad con ldap
  	+ homes, los usuarios locales ya lo tienen, los usuarios ldap no por tanto hay que creales el home, la propiedad y el grupo correcto.
   
#### smb.conf
```
[global]
        workgroup = MYGROUP
        server string = Samba Server Version %v
        log file = /var/log/samba/log.%m
        max log size = 50
        security = user
	passdb backend = ldapsam:ldap://ldap
          ldap suffix = dc=edt,dc=org
          ldap user suffix = ou=usuaris
          ldap group suffix = ou=grups
          ldap machine suffix = ou=hosts
          ldap idmap suffix = ou=domains
          ldap admin dn = cn=Manager,dc=edt,dc=org
          ldap ssl = no
          ldap passwd sync = yes  
        load printers = yes
	cups options = raw
[homes]
        comment = Home Directories
        browseable = no
        writable = yes
;       valid users = %S
;       valid users = MYDOMAIN\%S

```
#### smbldap.conf
```
# $Id$
#
# smbldap-tools.conf : Q & D configuration file for smbldap-tools

#  This code was developped by IDEALX (http://IDEALX.org/) and
#  contributors (their names can be found in the CONTRIBUTORS file).
#
#                 Copyright (C) 2001-2002 IDEALX
#
#  This program is free software; you can redistribute it and/or
#  modify it under the terms of the GNU General Public License
#  as published by the Free Software Foundation; either version 2
#  of the License, or (at your option) any later version.
#
#  This program is distributed in the hope that it will be useful,
#  but WITHOUT ANY WARRANTY; without even the implied warranty of
#  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#  GNU General Public License for more details.
#
#  You should have received a copy of the GNU General Public License
#  along with this program; if not, write to the Free Software
#  Foundation, Inc., 59 Temple Place - Suite 330, Boston, MA 02111-1307,
#  USA.

#  Purpose :
#       . be the configuration file for all smbldap-tools scripts

##############################################################################
#
# General Configuration
#
##############################################################################

# Put your own SID. To obtain this number do: "net getlocalsid".
# If not defined, parameter is taking from "net getlocalsid" return
#SID="S-1-5-21-2252255531-4061614174-2474224977"

# Domain name the Samba server is in charged.
# If not defined, parameter is taking from smb.conf configuration file
# Ex: sambaDomain="IDEALX-NT"
#sambaDomain="DOMSMB"

##############################################################################
#
# LDAP Configuration
#
##############################################################################

# Notes: to use to dual ldap servers backend for Samba, you must patch
# Samba with the dual-head patch from IDEALX. If not using this patch
# just use the same server for slaveLDAP and masterLDAP.
# Those two servers declarations can also be used when you have
# . one master LDAP server where all writing operations must be done
# . one slave LDAP server where all reading operations must be done
#   (typically a replication directory)

# Slave LDAP server URI
# Ex: slaveLDAP=ldap://slave.ldap.example.com/
# If not defined, parameter is set to "ldap://127.0.0.1/"
slaveLDAP="ldap://ldap/"

# Master LDAP server URI: needed for write operations
# Ex: masterLDAP=ldap://master.ldap.example.com/
# If not defined, parameter is set to "ldap://127.0.0.1/"
masterLDAP="ldap://ldap/"

# Use TLS for LDAP
# If set to 1, this option will use start_tls for connection
# (you must also used the LDAP URI "ldap://...", not "ldaps://...")
# If not defined, parameter is set to "0"
ldapTLS="0"

# How to verify the server's certificate (none, optional or require)
# see "man Net::LDAP" in start_tls section for more details
verify="require"

# CA certificate
# see "man Net::LDAP" in start_tls section for more details
cafile="/etc/pki/tls/certs/ldapserverca.pem"

# certificate to use to connect to the ldap server
# see "man Net::LDAP" in start_tls section for more details
clientcert="/etc/pki/tls/certs/ldapclient.pem"

# key certificate to use to connect to the ldap server
# see "man Net::LDAP" in start_tls section for more details
clientkey="/etc/pki/tls/certs/ldapclientkey.pem"

# LDAP Suffix
# Ex: suffix=dc=IDEALX,dc=ORG
suffix="dc=edt,dc=org"

# Where are stored Users
# Ex: usersdn="ou=Users,dc=IDEALX,dc=ORG"
# Warning: if 'suffix' is not set here, you must set the full dn for usersdn
usersdn="ou=usuaris,${suffix}"

# Where are stored Computers
# Ex: computersdn="ou=Computers,dc=IDEALX,dc=ORG"
# Warning: if 'suffix' is not set here, you must set the full dn for computersdn
computersdn="ou=hosts,${suffix}"

# Where are stored Groups
# Ex: groupsdn="ou=Groups,dc=IDEALX,dc=ORG"
# Warning: if 'suffix' is not set here, you must set the full dn for groupsdn
groupsdn="ou=grups,${suffix}"

# Where are stored Idmap entries (used if samba is a domain member server)
# Ex: idmapdn="ou=Idmap,dc=IDEALX,dc=ORG"
# Warning: if 'suffix' is not set here, you must set the full dn for idmapdn
idmapdn="ou=domains,${suffix}"

# Where to store next uidNumber and gidNumber available for new users and groups
# If not defined, entries are stored in sambaDomainName object.
# Ex: sambaUnixIdPooldn="sambaDomainName=${sambaDomain},${suffix}"
# Ex: sambaUnixIdPooldn="cn=NextFreeUnixId,${suffix}"
sambaUnixIdPooldn="sambaDomainName=${sambaDomain},${suffix}"

# Default scope Used
scope="sub"

# Unix password hash scheme (CRYPT, MD5, SMD5, SSHA, SHA, CLEARTEXT)
# If set to "exop", use LDAPv3 Password Modify (RFC 3062) extended operation.
password_hash="SSHA"

# if password_hash is set to CRYPT, you may set a salt format.
# default is "%s", but many systems will generate MD5 hashed
# passwords if you use "$1$%.8s". This parameter is optional!
password_crypt_salt_format="%s"

##############################################################################
# 
# Unix Accounts Configuration
# 
##############################################################################

# Login defs
# Default Login Shell
# Ex: userLoginShell="/bin/bash"
userLoginShell="/bin/bash"

# Home directory
# Ex: userHome="/home/%U"
userHome="/home/%U"

# Default mode used for user homeDirectory
userHomeDirectoryMode="700"

# Gecos
userGecos="System User"

# Default User (POSIX and Samba) GID
defaultUserGid="513"

# Default Computer (Samba) GID
defaultComputerGid="515"

# Skel dir
skeletonDir="/etc/skel"

# Treat shadowAccount object or not
shadowAccount="1"

# Default password validation time (time in days) Comment the next line if
# you don't want password to be enable for defaultMaxPasswordAge days (be
# careful to the sambaPwdMustChange attribute's value)
defaultMaxPasswordAge="45"

##############################################################################
#
# SAMBA Configuration
#
##############################################################################

# The UNC path to home drives location (%U username substitution)
# Just set it to a null string if you want to use the smb.conf 'logon home'
# directive and/or disable roaming profiles
# Ex: userSmbHome="\\PDC-SMB3\%U"
userSmbHome="\\PDC-SRV\%U"

# The UNC path to profiles locations (%U username substitution)
# Just set it to a null string if you want to use the smb.conf 'logon path'
# directive and/or disable roaming profiles
# Ex: userProfile="\\PDC-SMB3\profiles\%U"
userProfile="\\PDC-SRV\profiles\%U"

# The default Home Drive Letter mapping
# (will be automatically mapped at logon time if home directory exist)
# Ex: userHomeDrive="H:"
userHomeDrive="H:"

# The default user netlogon script name (%U username substitution)
# if not used, will be automatically username.cmd
# make sure script file is edited under dos
# Ex: userScript="startup.cmd" # make sure script file is edited under dos
userScript="logon.bat"

# Domain appended to the users "mail"-attribute
# when smbldap-useradd -M is used
# Ex: mailDomain="idealx.com"
mailDomain="example.com"

# Allows not to generate LANMAN password hash
lanmanPassword="0"

##############################################################################
#
# SMBLDAP-TOOLS Configuration (default are ok for a RedHat)
#
##############################################################################

# Allows not to use smbpasswd (if with_smbpasswd="0" in smbldap.conf) but
# prefer Crypt::SmbHash library
with_smbpasswd="0"
smbpasswd="/usr/bin/smbpasswd"

# Allows not to use slappasswd (if with_slappasswd="0" in smbldap.conf)
# but prefer Crypt:: libraries
with_slappasswd="0"
slappasswd="/usr/sbin/slappasswd"

# comment out the following line to get rid of the default banner
# no_banner="1"

```

#### smbldap_bind.conf

```
# $Id$
#
############################
# Credential Configuration #
############################
# Notes: you can specify two differents configuration if you use a
# master ldap for writing access and a slave ldap server for reading access
# By default, we will use the same DN (so it will work for standard Samba
# release)
slaveDN="cn=Manager,dc=edt,dc=org"
slavePw="secret"
masterDN="cn=Manager,dc=edt,dc=org"
masterPw="secret"
```
