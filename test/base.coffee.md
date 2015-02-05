    describe 'The module', ->
      it 'should compile', ->
        require '..'

      it 'should register with Knockout', ->
        ko = require 'knockout'
        (require '..') ko
