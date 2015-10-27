# bpg-ww
BPG JavaScript Decoder on WebWorkers

Allows 8 bit depth BPG images and animations

## Usage

```javascript
var bpgw = new Worker('/static/bpgdec8a-ww.min.js');
bpgw.onmessage = function(e) {
    switch(e.data.type) {
        case 'log': console.log('Worker log:' + e.data.message); break;
        case 'debug': console.log('Worker debug:' + e.data.data); debugger; break;
        case 'res':
            var img = e.data.image;
            var frames = e.data.frames;
            var loop_count = e.data.loop_count;
            var cnv = document.getElementById(e.data.meta);
            cnv.width = img.width;
            cnv.height = img.height;
            
            var ctx = cnv.getContext('2d');

            (function() {
                function d() {
                    var a = img.n;
                    ++a >= frames.length && (0 == loop_count || img.q < loop_count ? (a = 0, img.q++) : a =- 1);
                    0 <= a && (img.n = a, ctx.putImageData(frames[a].img, 0, 0), setTimeout(d, frames[a].duration))
                };
                ctx.putImageData(img, 0, 0);
                frames.length > 1 && (img.n = 0, img.q = 0, setTimeout(d, frames[0].duration));
            }.bind(ctx,img,frames,loop_count))();

            console.log('Decode done');
            break;
        default: console.log('Unknown event:' + e.data);
    };
};

// arrayBuffer: BPG image
// container: id of canvas (DOM) for rendering
function decodeBpg(arrayBuffer,container) {
    bpgw.postMessage({type:'image', img:arrayBuffer, meta:container});
};
```

### More info
* http://bellard.org/bpg/
