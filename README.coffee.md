A Knockout Widget for CCNQ(4) `rule` entry
------------------------------------------

This Knockout component manages a single `rule` entry as stored in a CCNQ4 `ruleset` database.
The layout of the record is adapted to the [`tough-rate`](https://github.com/shimaore/tough-rate) LCR routing engine.

A `rule` record must contain:
- _id: "rule:#{prefix}"
- type: "rule"
- prefix: prefix
- attrs: { cdr }
- gwlist: [ { ...}, {...}]
with possible gateways:
{ source_registrant:true }
{ carrierid }
{ gwid }

    module.exports = (ko) ->

      class RuleEntry
        constructor: ({doc,sip_domain_name,ruleset_db}) ->
          assert doc?, 'doc is required'
          assert ruleset_db?, 'ruleset_db is required'
          assert sip_domain_name?, 'sip_domain_name is required'
          assert doc?.prefix?, 'doc.prefix is required'
          assert doc?.attrs?.cdr?, 'doc.attrs.cdr is required'

Data
----

          @prefix = doc.prefix
          @cdr = doc.attrs.cdr
          @sip_domain_name = sip_domain_name
          @gwlist = ko.observableArray doc.gwlist

          @doc = doc
          @ruleset_db = ruleset_db

          @error = ko.observable ''

Behavior
--------

FIXME: I'm still quite confused about how KnockoutJS handles `this`, and why `add_gw` can be a simple object method, while it seems `remove_gw` needs to be an instance method.

          @remove_gw = (target) =>
            @gwlist.remove target
          @save = =>
            @error "Saving... (#{doc._rev})"
            doc._id ?= "rule:#{@prefix}"
            doc.type ?= 'rule'
            @ruleset_db.put @doc
            .then ({rev}) =>
              @doc._rev = rev
              @error 'Saved...'
            .catch (error) =>
              @error "Not saved: #{error} on #{JSON.stringify @doc}"

        add_gw: ->
          @gwlist.push {}

Layout
------

      html = ->
        {p,div,text,label,input,datalist,option,ul,li,button,tag} = teacup

        datalist '#gateway', bind: foreach: '$root.gateways', ->
          option bind: value: '$data'
        datalist '#carrier', bind: foreach: '$root.carriers', ->
          option bind: value: '$data'

        div ->
          p "Add a new prefix (routing)"
          label 'Prefix: '
          input
            type:'tel'
            bind:
              value: 'prefix'
            readonly:true
            required:true
          label 'Billing: '
          input
            bind:
              value: 'cdr'
            readonly:true
            required:true
        div ->
          text 'Targets: '
          div bind: foreach: 'gwlist', ->
            div ->

We use the custom `rule-target` component here.
FIXME: Is there a way to access `$root` from within the constructor of RuleTarget (in the ccnq-ko-rule-target project) instead of having to pass them along here? (Same question applies to `ruleset_db` in RuleEntry!)

              tag 'rule-target', params: 'data: $data, gateways: $root.gateways, carriers: $root.carriers'
              button bind: click: '$parent.remove_gw', 'Remove'
          button bind: click: 'add_gw', 'Add'
          button bind: click: 'save', 'Save'
        div '.error', bind: text: 'error', '(log)'

Extend Knockout witht the `rule-target` component/tag.

      (require 'ccnq-ko-rule-target') ko

Register the `rule-entry` component/tag.

      ko.components.register 'rule-entry',
        viewModel: RuleEntry
        template: teacup.render html

      RuleEntry

    teacup = require 'teacup'
    teacup.use (require 'teacup-databind')()
    assert = require 'assert'
