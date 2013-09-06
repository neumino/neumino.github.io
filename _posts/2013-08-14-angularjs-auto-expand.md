---
layout: post
category : Geek 
tags : [angularjs, directive, textarea, expand]
title: Auto expand textarea with AngularJS 
---
{% include JB/setup %}


While working on Chateau (data explorer for RethinkDB), I had to write a directive to auto expand a textarea to avoid scrollbars.
Since it may be useful for some people, here is the code:

The HTML code

```html
<textarea ng-auto-expand>What ever content you need</textarea>
```

The CSS code

```css
textarea{
    overflow: hidden;
    padding: 4px;
}
```


The directive

```js
angular.module('chateau.directives', [])
    .directive('ngAutoExpand', function() {
        return {
            restrict: 'A',
            link: function( $scope, elem, attrs) {
                elem.bind('keyup', function($event) {
                    var element = $event.target;

                    $(element).height(0);
                    var height = $(element)[0].scrollHeight;

                    // 8 is for the padding
                    if (height < 20) {
                        height = 28;
                    }
                    $(element).height(height-8);
                });

                // Expand the textarea as soon as it is added to the DOM
                setTimeout( function() {
                    var element = elem;

                    $(element).height(0);
                    var height = $(element)[0].scrollHeight;

                    // 8 is for the padding
                    if (height < 20) {
                        height = 28;
                    }
                    $(element).height(height-8);
                }, 0)
            }
        };
    });
```

