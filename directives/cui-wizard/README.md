# CUI Wizard
Version 1.0


### Description
Cui-wizard is an angular directive that, following a few syntax rules, allows the developer to easily create a multi-step form or any other component that requires a "step-through" approach.

### Usage Example

{% raw %}
```html
  <cui-wizard step="{{begginingStep}}" clickable-indicators minimum-padding="30" bar mobile-stack="700">
    <indicator-container></indicator-container>
    <step step-title="{{step1Title}}" state="{{stateName}}" icon="{{iconRef/Link}}">
      *step1 contents go here*
    </step>
    <step step-title="{{step2Title}}">
      *step2 contents go here*
    </step>
  </cui-wizard>
```
{% endraw %}

#### Variables
1. `beggining Step` this the step the wizard will start on.
2. `stepXTitle` the titles that will populate the indicator-container.
3. `stateName` the name of the state that the user is redirected to when he clicks that indicator, assuming he defined `clickable-indicators`.
4. `iconRef/Link` reference to the icon that will appear under the for the step. (look below for more info on how this works)


### How it works / features
The directive will start by reading the `step-title` atributes on each `step` element within the `cui-wizard`.
Then it creates step indicators (these are given `.step-indicator` class) which will be clickable depending on the presence of the `clickable-indicators` atribute.
The step that is currently active will give it's corresponding indicator an `.active` class.

As for the `icon`, if it contains a dot (.) the directive will interpret it as a link to an img and will create an img tag with a class of `.icon`. If it does not contain any dots it will automatically use the [`cui-icon directive`](https://github.com/thirdwavellc/cui-ng/tree/master/directives/cui-icon) to create an svg icon under the label, once again with an `.icon` class.

#### Changing steps
Anywhere inside of this directive you can position an element with an `ng-click` directive that calls one of 4 navigating functions:

*Note: variables in `<< >>` means they are optional. If << stateName >> is defined, it will also update the current state to whatever state is passed.

1. `next(<< nextStateName >>*)` -> increases the `step` attribute on cui-wizard by 1 and updates the wizard accordingly.
2. `nextWithErrorChecking(formName, << nextStateName >>*)` -> checks if every form field in that step is valid and will set a scope variable called `invalidForm[i]` (where i is the current step) to true if there are any errors. If there aren't errors, it simply calls `next()`. (you have to give your form and your inputs `name` attributes for this to work)
3. `goToStep(i)` -> navigates to a step with index i (note, steps start counting from 1)
4. `previous(<< previousStateName >>*)` -> navigates to the previous step
5. `goToState(state)` -> This is called automatically by `next`,`nextWithErrorChecking` and previous, if states are passed. What it does is use `$rootScope.$broadcast` to broadcast `'stepChange'` with {state:statePassed} as the data. This means you can use `$scope.$on` in your controller and listen for `'stepChange'`,like this:
```javascript
angular.module('app',['cui-ng','ui.router'])
.run(['$rootScope','wizardStep',function($rootScope,wizardStep){
  $rootScope.$on('$stateChangeStart', function(event, toState){
    if(toState.data && toState.data.step){
        wizardStep.set(toState.data.step);
        $rootScope.$broadcast('stepChange');
    }
  })
}])
.config(['$stateProvider','$urlRouterProvider','$injector',function($stateProvider,$urlRouterProvider,$injector){
  $stateProvider
    .state('wizard',{
        url: '/wizard',
        templateUrl: 'assets/angular-templates/cui-wizard.html',
        data: {
            step: 1
        }
    })
    .state('wizard.organization',{
        url: '/organization',
        data: {
            step: 1
        }
    })
    .state('wizard.signOn',{
        url: '/signOn',
        data: {
            step: 2
        }
    })
     .state('wizard.user',{
        url: '/user',
        data: {
            step: 3
        }
    })
    .state('wizard.review',{
        url: '/review',
        data: {
            step: 4
        }
    });

    //fixes infinite digest loop with ui-router
    $urlRouterProvider.otherwise( function($injector) {
      var $state = $injector.get("$state");
      $state.go('wizard');
    });
}])
.factory('wizardStep',[function(){
  var step;
  return{
      get: function(){
          return step;
      },
      set: function(newStep){
          step=newStep;
      }
  }
}])
.controller('appCtrl',['$scope','$state','wizardStep',function($scope,$state,wizardStep){
  $scope.$on('stepChange',function(e,data){
    if(data && data.state){
        app.goTo(data.state);
    }
    app.step=wizardStep.get();
  })

  app.goTo= function(state){
    $state.go(state,{notify:true,reload:true});
  }
}])
```

#### Key features
One of the key features of this wizard is indicator collision detection. What this means is:
Whenever the wizard is first shown or the window is rezised the directive will check that there is enough space within `indicator-container` to show all the step indicators AND the minimum padding (in px) between each of them, which is defined by `minimum-padding`.

If there isn't enough space then `indicator-container` gets applied a class of `.small` that can then be used to style accordingly with css. (tip: use this in conjunction with `.active` class on the indicators to give emphasis to the currently active step, by, for example, only showing the active step when there isn't enough room.

The `bar` attribute activates a bar with `.steps-bar` class within the `indicator-container`. Inside of this bar there will be another bar with a class of `.steps-bar-fill` that will increase in width based on the current step. (Note: Currently the bar grows from the middle of the 1st step indicator up to the middle of the last one)

Setting the `mobile-stack` attribute will use the `cui-expandable` directive to show the steps in an expandandable/collapsable manner. When in this mode, the expandables will have a class of `mobile-element`.

Cui-wizard will also listen for `'languageChange'` broadcasts on scope, and will fire the function that ensures there's enough room to show all of the indicators (and apply the class of `.small` to the `indicator-container` if there isn't). This is specifically built in for use with the [cui-i18n](https://github.com/thirdwavellc/cui-i18n) module.

## Change Log 9/20/2016

* Adds new optional attribute `dirty-validation`. This revolves around showing input field errors when they are `$dirty` instead
of the default of `$touched`. This also done not work with an `ng-messages-include=""`. This sets all input fields as `$dirty` when clicking the next button.

## Change Log 4/20/2016

* Major rework that enables us to not need to replicate the markup to display the mobile stack. Instead, we just use expandables with `transition-speed="0"` when in desktop mode and those same expandables with `transition-speed="300"` in mobile-stack mode. This will allow us to use `tether`, `cui-popover` and other tether based directives much more reliably, since these usually depened on an element id to use as the target.

## Change Log 3/28/2016

* Adds `.cui-steps` wrapper to the step-indicators to fix an issue where in IE and firefox the absolutely positioned bar would be taken into account for space-between.

## Change Log 2/2/2016

* Now only renders progress bar between steps if there's more than 1 step (you shouldn't be using this directive if you only have 1 step, but in the case that maybe you're dinamically assigning steps this might be useful)

## Change Log 1/29/2016

* New directive scope variable added - `wizardFinished` - true if the last step has been visited.

## Change Log 1/28/2016

* Correctly updates the step attribute on step change.

## Change Log 1/15/2016

* Now checks for `!form.$valid` rather than `form.$invalid` on `nextWithErrorChecking`.








