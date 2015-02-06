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
          @gwlist = ko.observableArray []
          if @doc.gwlist?
            for data in @doc.gwlist
              @gwlist.push new RuleTarget data
          return

We also provide the original document so that any other (extra) field is kept.

        update_doc: ->
          @doc._id = "rule:#{@prefix}"
          @doc.type = 'rule'
          @doc.prefix = @prefix
          @doc.gwlist = ko.toJS @gwlist
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

Add a new (empty) target.
FIXME: should select a default type based on existing targets (e.g. only one `source_registrant` makes sense).

        @add_gw = =>
          @gwlist.push new RuleTarget {}

Remove an existing target.

        @remove_gw = (target) =>
          @gwlist.remove target

Remove all invalid targets (so that what is shown on the UI is what will be saved)

        @clean_gwlist = =>
          for target in @gwlist()
            if not target._validated()
              @remove_gw target

        @save = =>
          @notify "Saving... (#{value._rev()})"

          @clean_gwlist()

          doc = value.update_doc()

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

      html: ({p,div,text,label,input,datalist,option,ul,li,button,tag}) ->

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

              rule_target '$data'
              button bind: click: '$parent.remove_gw', 'Remove'
          button bind: click: 'add_gw', 'Add'
          button bind: click: 'clean_gwlist', 'Cleanup'
          button bind: click: 'save', 'Save'
        div '.log', bind: text: 'notify', '(log)'

Extend Knockout witht the `rule-target` component/tag.

      {RuleTarget,rule_target} = (require 'ccnq-ko-rule-target') ko

    assert = require 'assert'
