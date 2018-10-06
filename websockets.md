NOTES

https://github.com/websocket-rails/websocket-rails

In `applicaiton.js` move `//= require websocket_rails/main` above `//=require_tree . `

As `//=require_tree . ` is the componenet that loads all the JS / Coffeescript files and if we're calling websockets in them, they need access to the websockets js lib.

Websockets needs a multi-threaded server

Thin is a good choice. Add 'Thin' or 'Unicon' into your gemfile

brew install ghostscript required for imagemagick.

Cropping to 400x300 from o,o in ImageMagick
	
	$ convert source.jpg -crop 400x300+0+0 output.jpg

re-size image to max 400 width or height and layer a mask on top.
	
	$ convert -resize 400x400\> -composite dog-portrait.png mask.png output.png

Add text to an image

re-size intial image, composit on a mask, write over it.


	convert -resize 400x400\>  -composite dog-portrait.png mask.png  -font 'Arial Narrow Bold Italic' -pointsize 18 -annotate +55+357 'Faerie Dragon' test.png
