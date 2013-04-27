<h1>Clown Car Technique for Responsive Images</h1>

<p>We can use media queries within SVG to serve up the right image. The beauty of the &quot;Clown Car&quot; technique is that all the logic remains in the SVG file. I've called it the &quot;Clown Car&quot; technique since we are including (or stuffing) many images (clowns) into a single image file (car).</p>
<p>When you mark up your HTML, you simply add a single call to an SVG file.</p>
<pre>&lt;img src=&quot;awesomefile.svg&quot; alt=&quot;responsive image&quot;&gt;</pre>
<p>Now isn't that code simple?</p>
<p>Unfortunately, due to content security policies (not CSP spec, but general security), FF and WebKit to not allow SVG files imported as &lt;img> to call in raster images or scripts. Therefore, we use the almost as simple &lt;object> tag, which as a non-empty element can provide for a child fallback for IE8 and Android 2.3.</p>
<pre>&lt;object data=&quot;awesomefile.svg&quot; &gt;&lt;/object></pre>
<p>The magic is that SVG supports both media queries and rasterized images. </p>
<p>In our SVG file, using the <code>&lt;image&gt;</code> element, will include the all the images that we may need to serve, and include all the media queries.</p>
<p>Here is the code for one of the SVG files:</p>
<pre>&lt;svg xmlns=&quot;http://www.w3.org/2000/svg&quot; xmlns:xlink=&quot;http://www.w3.org/1999/xlink&quot; width=&quot;300&quot; height=&quot;329&quot;&gt;
  &lt;title&gt;The Clown Car Technique&lt;/title&gt;
    &lt;defs&gt;
    &lt;style&gt;
    image {display: none;}
    #small {display: block}
     @media screen and (min-width: 401px) and (max-width: 700px) {
        #medium {display: block}
        #small {display: none}
    }
      @media screen and (min-width: 701px) and (max-width: 1000px) {
        #big {display: block}
        #small {display: none}
    }
     @media screen and (min-width: 1001px)  {
      #huge {display: block}
      #small {display: none;}
    }
    &lt;/style&gt;
  &lt;/defs&gt;
  &lt;g&gt;
    &lt;image id=&quot;small&quot;  height=&quot;329&quot; width=&quot;300&quot; xlink:href=&quot;images/small.png&quot; /&gt;
    &lt;image id=&quot;medium&quot; height=&quot;329&quot; width=&quot;300&quot; xlink:href=&quot;images/medium.png&quot; /&gt;
    &lt;image id=&quot;big&quot;    height=&quot;329&quot; width=&quot;300&quot; xlink:href=&quot;images/big.png&quot; /&gt;
    &lt;image id=&quot;huge&quot;   height=&quot;329&quot; width=&quot;300&quot; xlink:href=&quot;images/huge.png&quot; /&gt;
  &lt;/g&gt;
  &lt;/svg&gt;</pre>
<p>Unfortunately, when this file is used, all 4 PNGs are retrieved from the server. To solve this issue, we can use background images instead:</p>
<pre>
&lt;svg xmlns=&quot;http://www.w3.org/2000/svg&quot; viewBox=&quot;0 0 300 329&quot; preserveAspectRatio=&quot;xMidYMid meet&quot;&gt;
 &lt;title&gt;Clown Car Technique&lt;/title&gt;
 &lt;style&gt;
  svg {
    background-size: 100% 100%;
    background-repeat: no-repeat;
  }
 @media screen and (max-width: 400px) {
  svg {background-image: url(images/small.png&quot;);}
 }
 @media screen and (min-width: 401px) and (max-width: 700px) {
  svg {background-image: url(images/medium.png);}
 }
 @media screen and (min-width: 701px) and (max-width: 1000px) {
  svg {background-image: url(images/big.png);}
 }
 @media screen and (min-width: 1001px) {
  svg {background-image: url(images/huge.png);}
 }
 &lt;/style&gt;
&lt;/svg&gt;</pre>
<p>This version only downloads the images required, thereby solving the multi-HTTP and waste of bandwidth concerns. However, it seems to trigger more Content Security Policy issues than the previous SVG.</p>
<p>The SVG file has it's own declared size, but when included in HTML, the media query size is based on the proportions of the  DOM node in the HTML --. it reflects the space provided to it.</p>
<p>Open the <a href="http://estelle.github.io/clowncar/local.svg">SVG</a> in your browser and resize your browser window.  As the window grows and shrinks the image the SVG displays changes. The image size takes up the full screen because it has a background size of 100%. It's when we include the SVG in the &lt;object> of a flexible layout that the magic happens. You'll note that the first time you load  the SVG there may be flickers of white as the browser requests the next required PNG.</p>
<p>When you include the SVG in your HTML <code>&lt;img&gt;</code> with a flexible width, such as 70% of viewport, as you grow and shrink the container, the image responds to the changes. The &quot;width&quot;  media query in the SVG is based on the element width in which the SVG is contained, not the viewport width.</p>
<p> The foreground &lt;img> version works perfectly in Opera. In Chrome and Safari, I need to open the SVG file first, after which the HTML file containing the foreground SVG image works perfectly. In Firefox, the SVG works. Firefox  supports SVG and supports SVG as background image, but blocks the importing of external raster images due to their content security policy (CSP).</p>
<p>The content security policy does make sense: you don't want a file to be pulling in untrustworthy content. SVG technology is supported. It is just being prevented from pulling in external raster image. Firefox prevents it altogether. Safari and Chrome work if the SVG is preloaded. We can work with the browser vendors to get this CSP lifted, at least for same-origin files. </p>
<p>The browsers do all support the media queries with the SVG. They all support SVG as foreground or content images. They all support SVG as background images. The support just isn't all the same.</p>
<p>Responsive SVG for foreground images works. It simply works right out of the box. By using background images within the SVG itself the SVG only requests the file it needs. The raster images are background images. Only the required image is downloaded. The only negative is that it requires 2 http requests: one for the image and one for the SVG. I am currently working on getting SVG as data URI to work. I am not there yet.</p>
<p>Another pro for this technique: similar to how we separate content from presentation from behavior: this method enables us to also separate out images -- all the logic is in the SVG image instead of polluting our CSS or HTML.</p>
<p>The main question is: should we attempt this with &lt;object&gt;, or should we get browser vendors to change their content security policy to enable this technique to work as &lt;img> directly?</p>

<h2>Accessibility</h2>
<p>Object does not support the alt attribute, which make images more accessible. SVG can be made accessible with the help of the TITLE. So, don't forget the title as alt attribute in your SVG</p>