List Families
=============

A list can have from three parameters to many tens of them. Some listmasters need to create a set of lists that have the same profile. In order to simplify the apprehension of these parameters, list families define a lists typology. Families provide a new level for defaults : in the past, defaults in Sympa were global and most sites using Sympa needed multiple defaults for different group of lists. Moreover families allow listmaster to delegate a part of configuration list to owners, in a controlled way according to family properties. Distribution will provide defaults families.
 
Family concept
--------------

A family provides a model for all of its lists. It is specified by the following characteristics :

  - a list creation template providing a common profile for each list configuration file.
  - an degree of independence between the lists and the family : list parameters edition rights and constraints on these parameters can be *free* (no constraint), *controlled* (a set of available values is defined for these parameters) or *fixed* (the value for the parameter is imposed by the family). That prevents lists from diverging from the original and it allows list owner customizations in a controlled way.
  - a filiation kept between lists and family all along the list life : family modifications are applied on lists while keeping listowners customizations.

Here is a list of operation performed on a family :

  - definition : definition of the list creation template, the degree of independence and family customizations.
  - instantiation : lists creation or modifications of existing lists while respecting family properties. The set of data defining the lists is an XML document.
  - modification : modification of family properties. The modification is effective at the next instantiation time, that have consequences on every list.
  - closure : closure of each list.
  - adding one list to a family.
  - closing one family list.
  - modifying one family list.
 
Using family
------------

### Definition

Families can be defined at the robot level, at the site level or on the distribution level (where default families are provided). So, you have to create a sub directory named after the family's name in a `families` directory :

*Examples:*

``` code
/home/sympa/etc/families/my_family
/home/sympa/etc/my_robot/families/my_family
```

In this directory you must provide these files :

  - `config.tt2` (mandatory)
  - `param_constraint.conf` (mandatory)
  - `edit_list.conf`
  - customizable files

#### config.tt2

This is a list creation template, this file is mandatory. It provides default values for parameters. This file is an almost complete list configuration, with a number of missing fields (such as owner e-mail) to be replaced by data obtained at the time of family instantiation. It is easy to create new list templates by modifying existing ones. See [18.8](node19.html#list-tpl), page [![\[\*\]](crossref.png)](node19.html#list-tpl) and [17.1][(]customize/basics-templates.md#tpl-format), page [![\[\*\]](crossref.png)][(]customize/basics-templates.md#tpl-format).

*Example:*

``` code
subject [% subject %]

status [% status %]

[% IF topic %]
topics [% topic %]

[% END %]
visibility noconceal

send privateoreditorkey

web_archive
  access public

subscribe open_notify

shared_doc
  d_edit [% shared_edit %]
  d_read [% shared_read %]

lang [% language %]

[% FOREACH o = owner %]
owner
  email [% o.email %]
  profile privileged
  [% IF o.gecos %] 
  gecos [% o.gecos %]
  [% END %]

[% END %]
[% IF moderator %]
   [% FOREACH m = moderator %]
editor
  email [% m.email %]

   [% END %]
[% END %]
 
[% IF sql %]
include_sql_query
  db_type [% sql.type %]
  host [% sql.host %]
  user [% sql.user %]
  passwd [% sql.pwd %]
  db_name [% sql.name %]
  sql_query [% sql.query %]

[% END %]
ttl 360
```

#### param\_constraint.conf

This file is obligatory. It defines constraints on parameters. There are three kind of constraints :

  - *free* parameters : no constraint on these parameters, they are not written in the `param_constraint.conf` file.
  - *controlled* parameters : these parameters must select their values in a set of available values indicated in the `param_constraint.conf` file.
  - *fixed* parameters : these parameters must have the imposed value indicated in the `param_constraint.conf` file.

The parameters constraints will be checked at every list loading.

**WARNING** : Some parameters cannot be constrained, they are : `msg_topic.keywords` (see [21.4.13](node22.html#par-msg-topic), page [![\[\*\]](crossref.png)](node22.html#par-msg-topic)),`owner_include.source_parameter` (see [21.1.6](node22.html#par-owner-include), page [![\[\*\]](crossref.png)](node22.html#par-owner-include)), `editor_include.source_parameter` (see [21.1.2](node22.html#par-editor-include), page [![\[\*\]](crossref.png)](node22.html#par-editor-include)). About `digest` parameter (see [21.4.9](node22.html#par-digest), page [![\[\*\]](crossref.png)](node22.html#par-digest)) , just days can be constrained.

*Example:*

``` code
lang                fr,us           
archive.period      days,week,month 
visibility          conceal,noconceal   
shared_doc.d_read   public      
shared_doc.d_edit   editor
```

#### edit\_list.conf

This is an optional file. It defines which parameters/files are editable by owners. See [19.4.4][(]admin/list-creation.md#list-edition), page [![\[\*\]](crossref.png)][(]admin/list-creation.md#list-edition). If the family does not have this file, *Sympa* will look for the one defined on robot level, server site level or distribution level. (This file already exists without family context)
Notes that by default parameter family\_name is not writable, you should not change this edition right.

### customizable files

Families provides a new level of customization for scenarios (see [14][(]customize/basics-scenarios.md#scenarios), page [![\[\*\]](crossref.png)][(]customize/basics-scenarios.md#scenarios)), templates for service messages (see [17.2][(]customize/basics-templates.md#site-tpl), page [![\[\*\]](crossref.png)][(]customize/basics-templates.md#site-tpl)) and templates for web pages (see [17.3][(]customize/basics-templates.md#web-tpl) , page [![\[\*\]](crossref.png)][(]customize/basics-templates.md#web-tpl)). *Sympa* looks for these files in the following level order: list, family, robot, server site or distribution.

*Example of custom hierarchy :*

``` code
/usr/local/sympa-os/etc/families/myfamily/mail_tt2/
/usr/local/sympa-os/etc/families/myfamily/mail_tt2/bye.tt2
/usr/local/sympa-os/etc/families/myfamily/mail_tt2/welcome.tt2
```
 
### Instantiation

Instantiation permits to generate lists.You must provide an XML file that is composed of lists description, the root element is *family* and is only composed of *list* elements. List elements are described in section [19.1.2][(]admin/list-creation.md#xml-file-format), page [![\[\*\]](crossref.png)][(]admin/list-creation.md#xml-file-format). Each list is described by the set of values for affectation list parameters.

Here is an sample command to instantiate a family :

``` bash
sympa.pl --instantiate\_family my_family --robot \samplerobot --input\_file /path/to/my\_file.xml
```

This means lists that belong to family `my_family` will be created under the robot `my_robot` and these lists are described in the file `my_file.xml`. Sympa will split this file into several xml files describing lists. Each list XML file is put in each list directory.

*Example:*

``` xml
<?xml version="1.0" ?>
<family>
  <list>
    <listname>liste1</listname>
    <subject>a list example</subject>
    <description/>
    <status>open</status>
    <shared_edit>editor</shared_edit>
    <shared_read>private</shared_read>
    <language>fr</language>
    <owner multiple="1"> 
      <email>serge.aumont@cru.fr</email> 
      <gecos>C.R.U.</gecos>
    </owner>
    <owner multiple="1"> 
      <email>olivier.salaun@cru.fr</email>
    </owner>
    <owner_include multiple="1">
      <source>my_file</source>
    </owner_include>
    <sql> 
      <type>oracle</type>
      <host>sqlserv.admin.univ-x.fr</host>
      <user>stdutilisateur</user>
      <pwd>monsecret</pwd>
      <name>les_etudiants</name>
      <query>SELECT DISTINCT email FROM etudiant</query>
    </sql>
  </list>
  <list>
    <listname>liste2</listname>
    <subject>a list example</subject>
    <description/>
    <status>open</status>
    <shared_edit>editor</shared_edit>
    <shared_read>private</shared_read>
    <language>fr</language>
    <owner multiple="1"> 
      <email>serge.aumont@cru.fr</email> 
      <gecos>C.R.U.</gecos>
    </owner>
    <owner multiple="1"> 
      <email>olivier.salaun@cru.fr</email>
    </owner>
    <owner_include multiple="1">
      <source>my_file</source>
    </owner_include>
    <sql> 
      <type>oracle</type>
      <host>sqlserv.admin.univ-x.fr</host>
      <user>stdutilisateur</user>
      <pwd>monsecret</pwd>
      <name>les_etudiants</name>
      <query>SELECT DISTINCT email FROM etudiant</query>
    </sql>
  </list>
   ...
</family>
```

Each instantiation describes lists. Compared to the previous instantiation, there are three cases :

  - lists creation : new lists described by the new instantiation
  - lists modification : lists already existing but possibly changed because of changed parameters values in the XML file or because of changed family's properties.
  - lists removal : lists nomore described by the new instantiation. In this case, the listmaster must valid his choice on command line. If the list is removed, it is set in status `family_closed`, or if the list is recovered, the list XML file from the previous instantiation is got back to go on as a list modification then.

After list creation or modification, parameters constraints are checked :

  - *fixed* parameter : the value must be the one imposed.
  - *controlled* parameter : the value must be one of the set of available values.
  - *free* parameter : there is no checking.

diagram

In case of modification (see diagram), allowed customizations can be preserved :

  - (1) : for every modified parameters (via Web interface), noted in the `config_changes` file, values can be collected in the old list configuration file, according to new family properties :
      - *fixed* parameter : the value is not collected.
      - *controlled* parameter : the value is collected only if constraints are respected.
      - *free* parameter : the value is collected.
  - (2) : a new list configuration file is made with the new family properties
  - (3) : collected values are set in the new list configuration file.

Notes :

  - For each list problem (as family file error, error parameter constraint, error instanciation ...), the list is set in status `error_config` and the listmaster is notified. He will have to do necessary to put list in use.
  - For each list closing in family context, the list is set in status `family_closed` and the owner is notified.
  - For each overwritten list customization, the owner is notified.

### Modification

To modify a family, you have to edit family files manually. The modification will be effective while the next instanciation.
**WARNING**: The family modification must be done just before an instantiation. If it is not, alive lists wouldn't respect new family properties and they would be set in status error\_config immediately.

### Closure

Closes every list (installed under the indicated robot) of this family : lists status are set to `family_closed`, aliases are removed and subscribers are removed from DB. (a dump is created in the list directory to allow restoration of the list).

Here is a sample command to close a family :

``` bash
sympa.pl --close_family my_family --robot samplerobot
```

### Adding one list

Adds a list to the family without instantiate all the family. The list is created as if it was created during an instantiation, under the indicated robot. The XML file describes the list and the root element is `<list>`. List elements are described in section [19.3][(]admin/list-creation.md#list-creation-sympa), page [![\[\*\]](crossref.png)][(]admin/list-creation.md#list-creation-sympa).

Here is a sample command to add a list to a family :

``` bash
sympa.pl --add_list my_family --robot samplerobot  --input_file /path/to/my_file.xml
```

### Removing one list

Closes the list installed under the indicated robot : the list status is set to `family_closed`, aliases are removed and subscribers are removed from DB. (a dump is created in the list directory to allow restoring the list).

Here is a sample command to close a list family (same as an orphan list) :

``` bash
sympa.pl --close_list my_list@samplerobot
```
 
### Modifying one list

Modifies a family list without instantiating the whole family. The list (installed under the indicated robot) is modified as if it was modified during an instantiation. The XML file describes the list and the root element is `<list>`. List elements are described in section [19.3][(]admin/list-creation.md#list-creation-sympa), page [![\[\*\]](crossref.png)][(]admin/list-creation.md#list-creation-sympa).

Here is a sample command to modify a list to a family :

``` bash
sympa.pl --modify_list my_family --robot samplerobot --input_file /path/to/my_file.xml
```

### List parameters edition in a family context

According to file `edit_list.conf`, edition rights are controlled. See [19.4.4][(]admin/list-creation.md#list-edition), page [![\[\*\]](crossref.png)][(]admin/list-creation.md#list-edition). But in a family context, constraints parameters are added to edition right as it is summarized in this array :

array

Note : In order to preserve list customization for instantiation, every modified parameter (via the Web interface) is noted in the `config_changes` file.
 