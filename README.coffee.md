A Knockout Widget for CCNQ(4) `rule` entry
------------------------------------------

This Knockout component manages a single `rule` entry as stored in a CCNQ4 `ruleset` database.
The layout of the record is adapted to the [`tough-rate`](https://github.com/shimaore/tough-rate) LCR routing engine.

    module.exports = (require 'ccnq-ko') 'rule-entry', (ko) ->

Data
----

      @data class RuleEntry
        constructor: (@doc) ->
          assert @doc?, 'doc is required'
          assert @doc?.prefix?, 'doc.prefix is required'
          assert @doc?.attrs?.cdr?, 'doc.attrs.cdr is required'

          @prefix = @doc.prefix
          @cdr = ko.observable @doc.attrs.cdr
          @gwlist = new RuleGwlist @doc.gwlist
          return

We also provide the original document so that any other (extra) field is kept.

        update_doc: ->
          @doc._id = "rule:#{@prefix}"
          @doc.type = 'rule'
          @doc.prefix = @prefix
          @doc.gwlist = @gwlist.toJS()
          @doc.attrs ?= {}
          @doc.attrs.cdr = @cdr()
          @doc

Model
-----

      @view ({value,$root,ko}) ->
        assert value instanceof RuleEntry, 'value should be an instance of RuleEntry'
        {ruleset_db} = $root
        assert ruleset_db?, 'ruleset_db is required'

        @notify = ko.observable ''

        @save = =>
          doc = value.update_doc()

          @notify "Saving... (#{doc._rev})"

Save the record.

          ruleset_db.put doc
          .then ({rev}) =>
            doc._rev = rev
            @notify 'Saved...'
          .catch (error) =>
            @notify "Not saved: #{error} on #{JSON.stringify doc}"

        return

Layout
------

      @html ({p,div,text,label,input,datalist,option,ul,li,button,tag}) ->

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
          rule_gwlist 'gwlist'
          button bind: click: 'save', 'Save'
        div '.log', bind: text: 'notify', '(log)'

Extend Knockout witht the `rule-target` component/tag.

      {RuleGwlist,rule_gwlist} = (require 'ccnq-ko-rule-gwlist') ko

    assert = require 'assert'
