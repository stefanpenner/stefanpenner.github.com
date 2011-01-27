<link href='style.css' media='screen' rel='stylesheet' type='text/css' />

# Organize your JavaScript

The un-obtrusive JavaScript movement, has cleansed our views. No more are they a bazaar of disjoint in-lined JavaScript, but now our JavaScript resides in its appropriate library file(s). Optimized for maximum reuse and minimal duplication. 

But it seems this goal of a single global JavaScript file has some down sides. Often we trigger JavaScript based on which elements are on the given page. Discovering which elements are present, and which arent requires at least 1 DOM query. On smaller sites, where we poll for 2 or 3 things, this is not a bad thing. The issue arises when many such element lookups are needlessly performed per page.


Not only does this needlessly abuse the DOM, but also tends to prevent structured code. It seems many other have come to this conclusion. The real world example I will use is from ["jQuery.com"](http://static.jquery.com/files/rocker/scripts/custom.js). Now their ideas work well in preventing DOM abuse, but I feel may fall short when it comes to code organization.

    // JS file with url based conditional code execution 
    // (jquery.com)
    
    var loc = window.location.href;
    
    //Download
    if(loc.indexOf('http://jquery.com/Download') > -1){
      $('#jq-nav .jq-download').addClass('jq-current');
    }
    //Tutorials
    else if(loc.indexOf('http://jquery.com/Tutorials') > -1){
      $('#jq-nav .jq-tutorials').addClass('jq-current');
    }
    //UI within docs
    else if(loc.indexOf('http://jquery.com/UI') > -1){
      $('#jq-nav .jq-current').removeClass('jq-current');
      $('#jq-nav .jq-ui').addClass('jq-current');
    }
    
    ...// lots more

    else if(loc.indexOf('http://jquery.com/project') > -1){
      $('#jq-nav .jq-allPlugins').addClass('jq-current');
    }

Although the above is good practice, their are some commonalities that can be extracted. I believe a simple abstraction is possible, and can yield some cleaner, more conistent results. Let me suggest an experimental abstraction which might do the trick, [router.js](http://github.com/stefanpenner/router.js). The Following example is a port of the above jQuery example, using the new abstraction.


    // Obviously this Namespace is arbitrary
    //
    Iamstef.router(function(map){
    
      map.path(/(Download|Tutorials)/i, function(map,result){
        $('#jq-nav .jq-'+result[1]).addClass('jq-current');
      });;
    
      map.path(/UI/i, function(){
        $('#jq-nav .jq-current').removeClass('jq-current');
        $('#jq-nav .jq-ui').addClass('jq-current');
      });
    
      map.path(/project/i,function(){
        $('#jq-nav .jq-allPlugins').addClass('jq-current');
      });
    })();

So it seems the abstraction solves the above problem, in a nice clean slick and maybe elegant way, but is it enough? Is it useful enough to bother bundling it with your application? I believe yes, not necessarily for the above example, by abstracting some of the underlying  mechanisms away we get some other neat feature. Here are some examples.

    Iamstef.router(function(map){
      map.hostname(/orange/,function(map){
    
        // nested routing:
        //   this only executes if the outer condition has 
        //   been met.
        //
        map.hash(/apple/,function(map){
    
          // If the hostname matches orange, and the hash apple, 
          // then in this example the gallery also executes
          //
          gallery();
        });
      });
    
      // route and capture with the regex on the locations 
      // pathname:
      //   map: inner layer api call
      //   loc: the local location var, and in this cause
      //        its pathname attribute contains the matches 
      //        results of the regex.
      //
      map.pathname(/search=([^&]+)/,function(map,loc){
    
        // log the pathname resulting matches array, 
        // (if the match passes)
        //
        console.log(loc.pathname);
      });
    
      // route on the href
      //
      map.href(/google/,function(){
    
        // if the href matches google, execute the googlemap 
        //
        googlemap();
      });
    
      // route and match based on any of the location
      //  attributes
      //
      map.matches({ hostname : /google.com/,
                    href : /https?/},function(map){
    
        // mix and match
        //
        map.pathname(/1/,function(){});
      });
    })();

In addition, one can also capture the routing block, store it, and use it later, or trigger it repeatedly based on outside Events.

    var routes = Iamstef.router(function(map){}
      map.hash(/apple/,function(){ 
        ...
      });

      map.hash(/orange/,function(){
        
      };)
    });


At any point we can trigger the routers again by simply calling:

    routes();

It is also possible to send a custom location object in. Custom location object is for testing, or other use cases, such as rerouting based on the onHashChange event.

    var location = { ... };

    route(newLocation);

There you have it, a neat abstraction that if used provides some nice additional functionality. I have been using this in production for some time, and keep discovering new and useful use-cases. Now its your turn to try it out, please experiment, patch, fork, mutate to your hearts content.
