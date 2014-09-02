

# Tree Kit

This lib is a toolbox that provide functions to operate with nested `Object` structure.
It features the classic `.extend()` method, but provide a whole bunch of options that the others library lack.

* License: MIT
* Current status: beta
* Platform: Node.js only (browser support is planned)



# Install

Use Node Package Manager:

    npm install tree-kit



# Library references

In all examples below, it is assumed that you have required the lib into the `tree` variable:
```js
var tree = require( 'tree-kit' ) ;
```



## .extend( options , target , source1 , [source2] , [...] )

* options `Object` extend options, it supports the properties:
	* own `boolean` only copy owned properties from the sources
	* deep `boolean` perform a deep (recursive) extend
	* move `boolean` move properties from the sources object to the target object (delete properties from the sources object)
	* preserve `boolean` existing properties in the target object will not be overwritten
	* nofunc `boolean` skip properties that are functions
	* proto `boolean` alter the target's prototype so that it matches the source's prototype.
	  It forces option 'own'. Specifying multiple sources does not make sens here.
	* inherit `boolean` make the target inherit from the source (the target's prototype will be the source itself, not its prototype).
	  It forces option 'own' and disable 'proto'. Specifying multiple sources does not make sens here.
	* skipRoot `boolean` prevent the prototype of the target **root** object from mutation.
	  Only nested objects' prototype will be mutated.
	* deepBlacklist `Array` list of black-listed prototype: the recursiveness of the 'deep' option will be disabled for
	  object whose prototype is listed (only direct prototype will match, for performance purpose the rest of the
	  prototype chain will not be checked)
	* deepWhitelist `Array` the opposite of deepBlacklist, it's a white-list
	* flat `boolean|string` sources properties are copied in a way to produce a *flat* target, the target's key
	  is the full path (separated by '.') of the source's key, also if a string is provided it will be used as
	  the path separator
	* unflat `boolean|string` it is the opposite of 'flat': assuming that the sources are in the *flat* format,
	  it expands all flat properties -- whose name are path with '.' as the separator -- deeply into the target, 
	  also if a string is provided it will be used as the path separator
* target `Object` the target of the extend, properties will be copied to this object
* source `Object` the source of the extend, properties will be copied from this object

This is a full-featured *extend* of an object with one or more source object.

It is easily translated from jQuery-like *extend()*:
* `extend( target , source )` translate into `tree.extend( null , target , source )`
* `extend( true , target , source )` translate into `tree.extend( { deep: true } , target , source )`

However, here we have full control over what will be extended and how.

By default, `tree.extend()` will copy all enumerable properties, and perform a shallow copy (a nested object is not cloned, it remains
a reference of the original one).

With the *deep* option, a deep copy is performed, so nested object are cloned too.

The *own* option clone only owned properties from the sources, properties that are part of the source's prototype would not be copied/cloned.

You can also clone an object as close as it is possible to do in javascript by doing this:
```js
var clone = tree.extend( { deep: true, proto: true } , null , original ) ;
```
Also please note that:
* properties that are not enumerable will never be cloned: javascript does not provide a way to search for them
* design pattern using private members cannot be truly cloned since those private members are hidden in an inaccessible closure's scope

Mixing *inherit* and *deep* provides a nice multi-level inheritance.

With the *flat* option example:
```js
var o = {
	one: 1,
	sub: {
		two: 2,
		three: 3
	}
} ;

var flatCopy = tree.extend( { flat: true } , {} , o ) ;
```
... it will produce:
```js
{
	one: 1,
	"sub.two": 2,
	"sub.three": 3
}
```

By the way, the *unflat* option does the opposite, and thus can reverse this back to the original form.


## .diff( left , right , [options] )

* left `Object` the left-hand side object structure
* right `Object` the right-hand side object structure
* options `Object` containing options, it supports:
	* path `string` the initial path, default: empty string
	* pathSeparator `string` the path separator, default: '.'

This tool reports diff between a left-hand side and right-hand side object structure.
It returns an object, each key is a path where a difference is reported, the value being an object containing (again) the path
and a human-readable message.

See this example:
```js
var left = {
	a: 'a',
	b: 2,
	c: 'three',
	sub: {
		e: 5,
		f: 'six',
	}
} ;

var right = {
	b: 2,
	c: 3,
	d: 'dee',
	sub: {
		e: 5,
		f: 6,
	}
} ;

console.log( tree.diff( a , b ) ) ;
```
It will output:
```js
{ '.a': { path: '.a', message: 'does not exist in right-hand side' },
  '.c': { path: '.c', message: 'different typeof: string - number' },
  '.sub.f': { path: '.sub.f', message: 'different typeof: string - number' },
  '.d': { path: '.d', message: 'does not exist in left-hand side' } }
```





Full BDD spec generated by Mocha:


# TOC
   - [extend()](#extend)
   - [Diff](#diff)
   - [Masks](#masks)
   - [Inverse masks](#inverse-masks)
<a name=""></a>
 
<a name="extend"></a>
# extend()
should extend correctly an empty Object with a flat Object without depth (with or without the 'deep' option).

```js
var copy ;

var expected = {
	d : 4 ,
	e : undefined ,
	f : 3.14 ,
	g : 6 ,
	h : [] ,
	i : 'iii'
} ;

copy = tree.extend( { deep: true } , {} , input.subtree.subtree ) ;
expect( tree.extend( null , copy , input.subtree.subtree2 ) ).to.eql( expected ) ;

copy = tree.extend( { deep: true } , {} , input.subtree.subtree ) ;
expect( tree.extend( { deep: true } , copy , input.subtree.subtree2 ) ).to.eql( expected ) ;
```

should extend an empty Object with a deep Object performing a SHALLOW copy, the result should be equal to the deep Object, nested object MUST be equal AND identical.

```js
var copy = tree.extend( null , {} , input.subtree ) ;
expect( copy ).to.eql( input.subtree ) ;
expect( copy ).not.to.equal( input.subtree ) ;
expect( copy.subtree2 ).to.equal( input.subtree.subtree2 ) ;
```

with the 'deep' option should extend an empty Object with a deep Object performing a DEEP copy, the result should be equal to the deep Object, nested object MUST be equal BUT NOT identical.

```js
var copy = tree.extend( { deep: true } , {} , input.subtree ) ;
expect( copy ).to.eql( input.subtree ) ;
expect( copy ).not.to.equal( input.subtree ) ;
expect( copy.subtree2 ).not.to.equal( input.subtree.subtree2 ) ;
```

should extend (by default) properties of the prototype chain.

```js
var proto = {
	proto1: 'proto1' ,
	proto2: 'proto2' ,
} ;

var o = Object.create( proto ) ;

o.own1 = 'own1' ;
o.own2 = 'own2' ;

expect( tree.extend( null , {} , o ) ).to.eql( {
	proto1: 'proto1' ,
	proto2: 'proto2' ,
	own1: 'own1' ,
	own2: 'own2'
} ) ;

expect( tree.extend( { deep: true } , {} , o ) ).to.eql( {
	proto1: 'proto1' ,
	proto2: 'proto2' ,
	own1: 'own1' ,
	own2: 'own2'
} ) ;
```

with the 'own' option should ONLY extend OWNED properties, properties of the prototype chain are SKIPPED.

```js
var proto = {
	proto1: 'proto1' ,
	proto2: 'proto2' ,
} ;

var o = Object.create( proto ) ;

o.own1 = 'own1' ;
o.own2 = 'own2' ;

expect( tree.extend( { own: true } , {} , o ) ).to.eql( {
	own1: 'own1' ,
	own2: 'own2'
} ) ;

expect( tree.extend( { deep: true, own: true } , {} , o ) ).to.eql( {
	own1: 'own1' ,
	own2: 'own2'
} ) ;
```

with the 'deep' option should extend a deep Object into another deep Object correctly.

```js
var copy ;

copy = tree.extend( { deep: true } , {} , input.subtree ) ;
expect( tree.extend( null , copy , input.anotherSubtree ) ).to.eql( {
	a : 'A' ,
	b : 2 ,
	subtree: {
		l : '1one' ,
		m : false ,
		n : 'nay'
	} ,
	c : 'plusplus' ,
	subtree2: {
		p : true ,
		q : [4,5,6] ,
		r : '2'
	} ,
	j : 'Djay' ,
	k : 'ok' ,
	o : 'mg'
} ) ;

copy = tree.extend( { deep: true } , {} , input.subtree ) ;
expect( tree.extend( { deep: true } , copy , input.anotherSubtree ) ).to.eql( {
	a : 'A' ,
	b : 2 ,
	subtree: {
		d : 4 ,
		e : undefined ,
		f : 3.14 ,
		l : '1one' ,
		m : false ,
		n : 'nay'
	} ,
	c : 'plusplus' ,
	subtree2: {
		g : 6 ,
		h : [] ,
		i : 'iii',
		p : true ,
		q : [4,5,6] ,
		r : '2'
	} ,
	j : 'Djay' ,
	k : 'ok' ,
	o : 'mg'
} ) ;
```

with the 'proto' option and a null (or falsy) target, it should create and return a new Object with the prototype of the source Object.

```js
var e , o , proto ;

proto = {
	proto1: 'proto1' ,
	proto2: 'proto2' ,
	hello: function() { console.log( "Hello!" ) ; }
} ;

o = Object.create( proto ) ;
o.own1 = 'own1' ;
o.own2 = 'own2' ;

e = tree.extend( { proto: true } , null , o ) ;

expect( e ).not.to.be( o ) ;
expect( e.__proto__ ).to.equal( proto ) ;	// jshint ignore:line
expect( e ).to.eql( { own1: 'own1' , own2: 'own2' } ) ;
expect( e.proto1 ).to.be( 'proto1' ) ;
expect( e.proto2 ).to.be( 'proto2' ) ;
expect( typeof e.hello ).to.equal( 'function' ) ;
```

with the 'proto' option should change the prototype of each target properties for the prototype of the related source properties, if 'deep' is enabled it does so recursively.

```js
var e , o , proto1 , proto2 ;

proto1 = {
	proto1: 'proto1' ,
	hello: function() { console.log( "Hello!" ) ; }
} ;

proto2 = {
	proto2: 'proto2' ,
	world: function() { console.log( "World!" ) ; }
} ;

o = {
	own1: 'own1' ,
	own2: 'own2' ,
	embed1: Object.create( proto1 , { a: { value: 'a' , enumerable: true } } ) ,
	embed2: Object.create( proto2 , { b: { value: 'b' , enumerable: true } } )
} ;

e = tree.extend( { proto: true } , {} , o ) ;

expect( e ).not.to.be( o ) ;
expect( e ).to.eql( {
	own1: 'own1' ,
	own2: 'own2' ,
	embed1: { a: 'a' } ,
	embed2: { b: 'b' }
} ) ;
expect( e.embed1 ).to.be( o.embed1 ) ;
expect( e.embed2 ).to.be( o.embed2 ) ;
expect( e.embed1.proto1 ).to.be( 'proto1' ) ;
expect( e.embed2.proto2 ).to.be( 'proto2' ) ;
expect( typeof e.embed1.hello ).to.equal( 'function' ) ;
expect( typeof e.embed2.world ).to.equal( 'function' ) ;


e = tree.extend( { proto: true, deep: true } , {} , o ) ;

expect( e ).not.to.be( o ) ;
expect( e ).to.eql( {
	own1: 'own1' ,
	own2: 'own2' ,
	embed1: { a: 'a' } ,
	embed2: { b: 'b' }
} ) ;
expect( e.embed1 ).not.to.be( o.embed1 ) ;
expect( e.embed2 ).not.to.be( o.embed2 ) ;
expect( e.embed1.proto1 ).to.be( 'proto1' ) ;
expect( e.embed2.proto2 ).to.be( 'proto2' ) ;
expect( typeof e.embed1.hello ).to.equal( 'function' ) ;
expect( typeof e.embed2.world ).to.equal( 'function' ) ;
```

with 'nofunc' option should skip function.

```js
var e , o , proto ;

proto = {
	proto1: 'proto1' ,
	proto2: 'proto2' ,
	hello: function() { console.log( "Hello..." ) ; }
} ;

o = Object.create( proto ) ;
o.own1 = 'own1' ;
o.world = function() { console.log( "world!!!" ) ; } ;
o.own2 = 'own2' ;

// default behaviour
e = tree.extend( { nofunc: true } , null , o ) ;
expect( e ).not.to.be( o ) ;
expect( e ).to.eql( { own1: 'own1' , own2: 'own2' , proto1: 'proto1' , proto2: 'proto2' } ) ;

// with 'own'
e = tree.extend( { nofunc: true , own: true } , null , o ) ;
expect( e ).not.to.be( o ) ;
expect( e ).to.eql( { own1: 'own1' , own2: 'own2' } ) ;

// with 'proto', function exists if there are in the prototype
e = tree.extend( { nofunc: true , proto: true } , null , o ) ;
expect( e ).not.to.be( o ) ;
expect( e.__proto__ ).to.equal( proto ) ;	// jshint ignore:line
expect( e ).to.eql( { own1: 'own1' , own2: 'own2' } ) ;
expect( e.proto1 ).to.be( 'proto1' ) ;
expect( e.proto2 ).to.be( 'proto2' ) ;
expect( typeof e.hello ).to.equal( 'function' ) ;
```

with 'preserve' option should not overwrite existing properties in the target.

```js
var e , o ;

e = {
	one: '1' ,
	two: 2 ,
	three: 'THREE'
} ;

o = {
	three: 3 ,
	four: '4'
} ;

tree.extend( { preserve: true } , e , o ) ;
expect( e ).to.eql( { one: '1' , two: 2 , three: 'THREE' , four: '4' } ) ;
expect( o ).to.eql( { three: 3 , four: '4' } ) ;
```

with 'move' option should move source properties to target properties, i.e. delete them form the source.

```js
var e , o ;

e = {
	one: '1' ,
	two: 2 ,
	three: 'THREE'
} ;

o = {
	three: 3 ,
	four: '4'
} ;

tree.extend( { move: true } , e , o ) ;
expect( e ).to.eql( { one: '1' , two: 2 , three: 3 , four: '4' } ) ;
expect( o ).to.eql( {} ) ;
```

with 'preserve' and 'move' option should not overwrite existing properties in the target, so it should not move/delete them from the source object.

```js
var e , o ;

e = {
	one: '1' ,
	two: 2 ,
	three: 'THREE'
} ;

o = {
	three: 3 ,
	four: '4'
} ;

tree.extend( { preserve: true , move: true } , e , o ) ;
expect( e ).to.eql( { one: '1' , two: 2 , three: 'THREE' , four: '4' } ) ;
expect( o ).to.eql( { three: 3 } ) ;
```

with 'inherit' option should inherit rather than extend: each source property create a new Object or mutate existing Object into the related target property, using itself as the prototype.

```js
var e , o ;

o = {
	three: 3 ,
	four: '4' ,
	subtree: {
		five: 'FIVE' ,
		six: 6
	}
} ;

e = {} ;

tree.extend( { inherit: true } , e , o ) ;

expect( e.__proto__ ).to.equal( o ) ;	// jshint ignore:line
expect( e ).to.eql( {} ) ;
expect( e.three ).to.be( 3 ) ;
expect( e.four ).to.be( '4' ) ;
expect( e.subtree ).to.equal( o.subtree ) ;


e = {
	one: '1' ,
	two: 2 ,
	three: 'THREE' ,
} ;

tree.extend( { inherit: true } , e , o ) ;

expect( e.__proto__ ).to.equal( o ) ;	// jshint ignore:line
expect( e ).to.eql( { one: '1' , two: 2 , three: 'THREE' } ) ;
expect( e.three ).to.be( 'THREE' ) ;
expect( e.four ).to.be( '4' ) ;
expect( e.subtree ).to.equal( o.subtree ) ;	// jshint ignore:line
expect( e.subtree ).to.eql( { five: 'FIVE' , six: 6 } ) ;


e = {
	one: '1' ,
	two: 2 ,
	three: 'THREE' ,
	subtree: {
		six: 'SIX' ,
		seven: 7
	}
} ;

tree.extend( { inherit: true } , e , o ) ;

expect( e.__proto__ ).to.equal( o ) ;	// jshint ignore:line
expect( e ).to.eql( { one: '1' , two: 2 , three: 'THREE' , subtree: { six: 'SIX' , seven: 7 } } ) ;
expect( e.three ).to.be( 'THREE' ) ;
expect( e.four ).to.be( '4' ) ;
expect( e.subtree ).to.eql( { six: 'SIX' , seven: 7 } ) ;
expect( e.subtree.five ).to.equal( undefined ) ;
```

with 'inherit' and 'deep' option should inherit recursively.

```js
var e , o ;

o = {
	three: 3 ,
	four: '4' ,
	subtree: {
		five: 'FIVE' ,
		six: 6
	}
} ;

e = {} ;

tree.extend( { inherit: true , deep: true } , e , o ) ;

expect( e.__proto__ ).to.equal( o ) ;	// jshint ignore:line
expect( e ).to.eql( { subtree: {} } ) ;
expect( e.three ).to.be( 3 ) ;
expect( e.four ).to.be( '4' ) ;
expect( e.subtree.__proto__ ).to.equal( o.subtree ) ;	// jshint ignore:line
expect( e.subtree.five ).to.equal( 'FIVE' ) ;
expect( e.subtree.six ).to.equal( 6 ) ;


e = {
	one: '1' ,
	two: 2 ,
	three: 'THREE' ,
} ;

tree.extend( { inherit: true , deep: true } , e , o ) ;

expect( e.__proto__ ).to.equal( o ) ;	// jshint ignore:line
expect( e ).to.eql( { one: '1' , two: 2 , three: 'THREE' , subtree: {} } ) ;
expect( e.three ).to.be( 'THREE' ) ;
expect( e.four ).to.be( '4' ) ;
expect( e.subtree.__proto__ ).to.equal( o.subtree ) ;	// jshint ignore:line
expect( e.subtree ).to.eql( {} ) ;
expect( e.subtree.five ).to.equal( 'FIVE' ) ;
expect( e.subtree.six ).to.equal( 6 ) ;


e = {
	one: '1' ,
	two: 2 ,
	three: 'THREE' ,
	subtree: {
		six: 'SIX' ,
		seven: 7
	}
} ;

tree.extend( { inherit: true , deep: true } , e , o ) ;

expect( e.__proto__ ).to.equal( o ) ;	// jshint ignore:line
expect( e ).to.eql( { one: '1' , two: 2 , three: 'THREE' , subtree: { six: 'SIX' , seven: 7 } } ) ;
expect( e.three ).to.be( 'THREE' ) ;
expect( e.four ).to.be( '4' ) ;
expect( e.subtree.__proto__ ).to.equal( o.subtree ) ;	// jshint ignore:line
expect( e.subtree ).to.eql( { six: 'SIX' , seven: 7 } ) ;
expect( e.subtree.five ).to.equal( 'FIVE' ) ;
```

with 'flat' option.

```js
var e , o ;

o = {
	three: 3 ,
	four: '4' ,
	subtree: {
		five: 'FIVE' ,
		six: 6 ,
		subsubtree: {
			subsubsubtree: { one: 'ONE' } ,
			seven: 'seven'
		} ,
		emptysubtree: {}
	} ,
	eight: 8 ,
	anothersubtree: {
		nine: '9'
	}
} ;

e = tree.extend( { flat: true } , {} , o ) ;
expect( e ).to.eql( {
	three: 3 ,
	four: '4' ,
	'subtree.five': 'FIVE' ,
	'subtree.six': 6 ,
	'subtree.subsubtree.seven': 'seven' ,
	'subtree.subsubtree.subsubsubtree.one': 'ONE' ,
	eight: 8 ,
	'anothersubtree.nine': '9'
} ) ;

e = tree.extend( { flat: '/' } , {} , o ) ;
expect( e ).to.eql( {
	three: 3 ,
	four: '4' ,
	'subtree/five': 'FIVE' ,
	'subtree/six': 6 ,
	'subtree/subsubtree/seven': 'seven' ,
	'subtree/subsubtree/subsubsubtree/one': 'ONE' ,
	eight: 8 ,
	'anothersubtree/nine': '9'
} ) ;
```

with 'unflat' option.

```js
var e , o ;

o = {
	three: 3 ,
	four: '4' ,
	'subtree.five': 'FIVE' ,
	'subtree.six': 6 ,
	'subtree.subsubtree.seven': 'seven' ,
	'subtree.subsubtree.subsubsubtree.one': 'ONE' ,
	eight: 8 ,
	'anothersubtree.nine': '9'
} ;

e = tree.extend( { unflat: true } , {} , o ) ;
expect( e ).to.eql( {
	three: 3 ,
	four: '4' ,
	subtree: {
		five: 'FIVE' ,
		six: 6 ,
		subsubtree: {
			subsubsubtree: { one: 'ONE' } ,
			seven: 'seven'
		}
	} ,
	eight: 8 ,
	anothersubtree: {
		nine: '9'
	}
} ) ;

o = {
	three: 3 ,
	four: '4' ,
	'subtree/five': 'FIVE' ,
	'subtree/six': 6 ,
	'subtree/subsubtree/seven': 'seven' ,
	'subtree/subsubtree/subsubsubtree/one': 'ONE' ,
	eight: 8 ,
	'anothersubtree/nine': '9'
} ;

e = tree.extend( { unflat: '/' } , {} , o ) ;
expect( e ).to.eql( {
	three: 3 ,
	four: '4' ,
	subtree: {
		five: 'FIVE' ,
		six: 6 ,
		subsubtree: {
			subsubsubtree: { one: 'ONE' } ,
			seven: 'seven'
		}
	} ,
	eight: 8 ,
	anothersubtree: {
		nine: '9'
	}
} ) ;
```

<a name="diff"></a>
# Diff
should return an array of differences for two objects without nested object.

```js
var a = {
	a: 'a',
	b: 2,
	c: 'three'
} ;

var b = {
	b: 2,
	c: 3,
	d: 'dee'
} ;

var diff = tree.diff( a , b ) ;

//console.log( diff ) ;
expect( diff ).not.to.be( null ) ;
expect( diff ).to.only.have.keys( '.a', '.c', '.d' ) ;
```

should return an array of differences for two objects with nested objects.

```js
var a = {
	a: 'a',
	b: 2,
	c: 'three',
	sub: {
		e: 5,
		f: 'six',
		subsub: {
			g: 'gee',
			h: 'h'
		}
	},
	suba: {
		j: 'djay'
	}
} ;

var b = {
	b: 2,
	c: 3,
	d: 'dee',
	sub: {
		e: 5,
		f: 6,
		subsub: {
			g: 'gee',
			i: 'I'
		}
	},
	subb: {
		k: 'k'
	}
} ;

var diff = tree.diff( a , b ) ;

//console.log( diff ) ;
expect( diff ).not.to.be( null ) ;
expect( diff ).to.only.have.keys( '.a', '.c', '.d', '.sub.f', '.sub.subsub.h', '.sub.subsub.i', '.suba', '.subb' ) ;
```

<a name="masks"></a>
# Masks
should apply a simple mask tree to the input tree.

```js
var mask = tree.createMask( {
	int: true,
	float: true,
	attachement: {
		filename: true,
		unexistant: true
	},
	unexistant: true,
	subtree: {
		subtree: true
	}
} ) ;

var output = mask.applyTo( input ) ;

expect( output ).to.eql( {
	int: 12,
	float: 2.47,
	attachement: {
		filename: 'preview.png'
	},
	subtree: {
		subtree: {
			d: 4,
			e: undefined,
			f: 3.14
		}
	}
} ) ;
```

should apply a mask tree with wildcard '*' to the input tree.

```js
var mask = tree.createMask( {
	'files': {
		'*': {
			size: true,
			unexistant: true
		}
	}
} ) ;

var output = mask.applyTo( input ) ;

expect( output.files ).to.be.an( Object ) ;
expect( output ).to.eql( {
	files: {
		'background.png' : {
			size : '97856'
		} ,
		'header.png' : {
			size : '44193'
		} ,
		'footer.png' : {
			size : '36411'
		}
	}
} ) ;
```

should apply a mask tree with wildcard '*' to match array in the input tree.

```js
var mask = tree.createMask( {
	'filesArray': {
		'*': {
			name: true,
			size: true,
			unexistant: true
		}
	}
} ) ;

var output = mask.applyTo( input ) ;

expect( output.filesArray ).to.be.an( Array ) ;
expect( output ).to.eql( {
	filesArray: [
		{
			name : 'background.png' ,
			size : '97856'
		} ,
		{
			name : 'header.png' ,
			size : '44193'
		} ,
		{
			name : 'footer.png' ,
			size : '36411'
		}
	]
} ) ;

//console.log( "\n\n\n\n" , output , "\n\n\n\n" ) ;
```

should apply a mask with a mask's leaf callback to the input tree.

```js
var leaf = function leaf( input , key , argument , path ) {
	//console.log( 'LEAF: ' , input , key , argument , path ) ;
	
	if ( ! input.hasOwnProperty( key ) ) { return new Error( 'not_found' ) ; }
	if ( typeof input[ key ] === 'number' ) { return input[ key ] + argument ; }
	return input[ key ] ;
} ;

var mask = tree.createMask(
	{
		int: 87 ,
		float: 14 ,
		subtree: {
			subtree: {
				f: 0.0016
			}
		} ,
		unexistant: 45
	} ,
	{ leaf: leaf }
) ;

var output = mask.applyTo( input ) ;

expect( output ).to.eql( {
	int: 99,
	float: 16.47,
	subtree: {
		subtree: {
			f: 3.1416
		}
	}
} ) ;
```

should apply a mask containing other masks to the input tree.

```js
var mask = tree.createMask( {
	int: true,
	float: true,
	attachement: tree.createMask( {
		filename: true,
		unexistant: true
	} ),
	unexistant: true,
	subtree: tree.createMask( {
		subtree: true
	} )
} ) ;

var output = mask.applyTo( input ) ;

expect( output ).to.eql( {
	int: 12,
	float: 2.47,
	attachement: {
		filename: 'preview.png'
	},
	subtree: {
		subtree: {
			d: 4,
			e: undefined,
			f: 3.14
		}
	}
} ) ;
```

<a name="inverse-masks"></a>
# Inverse masks
should apply a simple mask tree to the input tree.

```js
var mask = tree.createInverseMask( {
	a: true,
	subtree: {
		d: true
	},
	subtree2: true
} ) ;

var output = mask.applyTo( input.subtree ) ;

//console.log( output ) ;

expect( output ).to.eql( {
	b: 2,
	subtree: {
		e: undefined,
		f: 3.14
	},
	c: 'plusplus'
} ) ;
```

