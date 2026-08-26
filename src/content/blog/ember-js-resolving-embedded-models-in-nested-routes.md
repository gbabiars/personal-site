---
title: "Ember.js - Resolving Embedded Models in Nested Routes"
description: "I recently ran into a roadblock with Ember and Ember Data where I was trying to access an embedded model for a nested route. What I wanted to do was return one of the elements…"
pubDate: "2013-08-10T00:00:00"
---

I recently ran into a roadblock with Ember and Ember Data where I was trying to access an embedded model for a nested route. What I wanted to do was return one of the elements of an array in the model from the parent route as my model for the nested route. The solution ended up being fairly simple.

My route definitions looked as such:

    App.Router.map(function() {
        // ...
        this.resource('editGame', { path: '/editable-games/edit/:editableGame_id' }, function() {
            this.resource('editGameRound', { path: '/:round' });
        });
    });

And my models were defined as:

    App.EditableGame = DS.Model.extend({
        rounds: DS.hasMany('App.EditableBoard'),
        // ...
    });

    App.EditableBoard = DS.Model.extend({
        round: DS.attr('number'),
        // ...
    });

It was easy enough to load the model for the EditGame route, but for the EditGameRound route I wanted to return one of the rounds from the parent EditGame model based on which round was passed into the route.

The model function on Ember.Route gives us the Transition object as a second parameter. From this transition object, we have access to "resolvedModels" which includes the resolved model from our parent route. So to get access to the embedded EditableBoard from the EditableGame model, we just need to access the resolved model by route name:

    var roundNames = {
        1: 'first',
        2: 'second'
    };

    App.EditGameRoundRoute = Em.Route.extend({
        serialize: function(model) {
            return { round: roundNames[model.get('round')] ||'final' };
        },

        model: function(params, transition) {
            var game = transition.resolvedModels.editGame,
                round = params.round;
            if(round === 'first') {
                return game.get('rounds').objectAt(0);
            }
            if(round === 'second') {
                return game.get('rounds').objectAt(1);
            }
        }
    });

Note that I accessed the game by the parent route's name (editGame). This made it very simple to find and return my embedded model and return it as the model of the nested route.
