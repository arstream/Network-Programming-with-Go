## Other Bits: JavaScript and CSS

 On request, a set of flashcards will be loaded into the browser. A much abbreviated set is shown below. The display of these cards is controlled by JavaScript and CSS files. These aren't relevant to the Go server so are omitted. Those interested can download the code.

```html
<html>
  <head>
    <title>
      Flashcards for Common Words
    </title>

    <link type="text/css" rel="stylesheet" 
          href="/html/CardStylesheet.css">
    </link>

    <script type="text/javascript" 
            language="JavaScript1.2" src="/jscript/jquery.js">
      <!-- empty -->
    </script>

    <script type="text/javascript" 
            language="JavaScript1.2" src="/jscript/slideviewer.js">
      <!-- empty -->
    </script>

    <script type="text/javascript" 
            language="JavaScript1.2">
      cardOrder = RANDOM;
      showHalfCard = RANDOM_HALF;
    </script>
  </head>
  <body onload="showSlides();"> 
    <h1> 
      Flashcards for Common Words
    </h1>
    <p>
        <div class="card">

	  <div class="english">
	    <div class="vcenter">
	      hello
	    </div>
	  </div>

	      <div class="pinyin">
		<div class="vcenter">
		  nǐ hǎo
		</div>

	      </div>
	      
	      <div class="traditional">
		<div class="vcenter">
		  你好
		</div>
	      </div>
	      
	      <div class="simplified">
		<div class="vcenter">
		  你好
		</div>

	      </div>

	      <div class ="translations">
		<div class="vcenter">
		  hello <br />
		  hi <br />
		  how are you? <br />
		</div>

              </div>
        </div>
        <div class="card">
	  <div class="english">
	    <div class="vcenter">
	      hello (interj., esp. on telephone)
	    </div>
	  </div>

	      <div class="pinyin">

		<div class="vcenter">
		  wèi
		</div>
	      </div>
	      
	      <div class="traditional">
		<div class="vcenter">
		  喂
		</div>
	      </div>
	      
	      <div class="simplified">

		<div class="vcenter">
		  喂
		</div>
	      </div>

	      <div class ="translations">
		<div class="vcenter">
		  hello (interj., esp. on telephone) <br />
		  hey <br />

		  to feed (sb or some animal) <br />
		</div>
              </div>
        </div>
    </p>

    <p class ="return">
      Press &lt;Space&gt; to continue
	<br/>	
      <a href="http:/flashcards.html"> Return to Flash Cards list</a>
    </p>
  </body>
</html>
```


