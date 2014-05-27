newsQuery
=========

The newsQuery module provides easy-to-consume access to the BBC News Labs APIs.

The APIs let you query a database of 40+ news sources, including content from the BBC but also other publications including The Guardian, The Mirror, The Huffington Post and others.

The majority of our content is articles but we also have images, video and tweets from a select range of sources.

The BBC News Labs API's are experimental and are cheifly intended for use by R&D teams in news orgs and in universities. If you'd like to more more or have any feedback about them, you can get in touch with @BBC_News_Labs on Twitter.

**Important!** To use this library you must have a BBC News Labs API key (which is free). See instructions below for how to do this.

### How to get an API key

You can obtain an API key from the BBC Developer Portal
http://bbc.apiportal.apigee.com

Registration is free and immediate, you will receive an automated email when you sign up, which contains a link to activate your account. Check your junk mail folder if you can't find it.

1. After registering, create a new application.
2. Select both the "bbcrd-newslabs-apis-product" and "bbcrd-juicer-apis-product" APIs for your application.
3. Your API key will be listed as the "Consumer Key" for your application.

## Additional documentation

You can find full information about how the BBC News Labs semantic APIs work on the #newsHACK site:

The semantic News Labs API:
http://newshack.co.uk/newshack-ii/newslabs-apis/

The News Juicer API:
http://newshack.co.uk/newshack-ii/juicer-apis/

## Usage with examples

### getConcepts()

You need to get a definative URI for a concept before you can search for articles that mention it.

A concept is typically a person, place, organisation or theme (e.g. "law", "economics"). These correspond to entries in dbpedia, which uses an ontology derived from data in Wikipedia.

An example that returns the first 5 concepts matching the term "Rooney":

``` javascript
var apiKey = '1234567890ABCDEF';
var newsQuery = require('newsquery')(apiKey);
newsQuery.getConcepts("Rooney", 5)
.then(function(concepts) {
    console.log(concepts);
});
```

The response from getConcepts() is an array of concepts, each with a unique URI (and possibly an image).

Note in the example below 'Wayne Rooney' is a SoccerPlayer, and SoccerPlayers are Atheletes, and Atheletess are People.

Mickey Rooney is classed as a Person, although it has not specifically identified him as an Actor - the granularity of the specificify may vary from concept to concept.

NB: 'Agent', which is as the last type for both, is simply a top level ontology class associated with all People and Organisations (and does not refer to being sports or acting agent). In this context an 'Agent' is a thing that acts in the world, for example, as distinct from something that is an Activity or a Place.

``` javascript
[ { name: 'Wayne Rooney',
    uri: 'http://dbpedia.org/resource/Wayne_Rooney',
    image: 'http://upload.wikimedia.org/wikipedia/commons/thumb/1/11/Rooney_CL.jpg/200px-Rooney_CL.jpg',
    type: 
     [ 'http://dbpedia.org/ontology/SoccerPlayer',
       'http://dbpedia.org/ontology/Athlete',
       'http://dbpedia.org/ontology/Person',
       'http://dbpedia.org/ontology/Agent' ] },
  { name: 'Mickey Rooney',
    uri: 'http://dbpedia.org/resource/Mickey_Rooney',
    image: 'http://upload.wikimedia.org/wikipedia/commons/thumb/c/c5/Mickey_Rooney_still.jpg/200px-Mickey_Rooney_still.jpg',
    type: 
     [ 'http://dbpedia.org/ontology/Person',
       'http://dbpedia.org/ontology/Agent' ] }
]
````

### getConcept()

If you have the definative URI for a concept you can use it's URI to request additional information about it.

For example, you can check the "type" field to see if it's categorized as a Person, Place, Organisation, etc (you can also check for more specific types, like SoccerPlayer or Company).

``` javascript
var apiKey = '1234567890ABCDEF';
var newsQuery = require('newsquery')(apiKey);
newsQuery.getConcept("http://dbpedia.org/resource/David_Cameron", 1)
.then(function(concept) {
    console.log(concept);
});
```

The response is a single object, with a description of it

``` javascript
 { description: 'David William Donald Cameron is the Prime Minister of the United Kingdom, First Lord of the Treasury, Minister for the Civil Service and Leader of the Conservative Party. He represents Witney as its Member of Parliament (MP). Cameron studied Philosophy, Politics and Economics (PPE) at Oxford, gaining a first class honours degree. He then joined the Conservative Research Department and became Special Adviser to Norman Lamont, and then to Michael Howard.',
  articles: 
   [ { product: 'http://www.bbc.co.uk/ontologies/bbc/IrishIndependant',
       summary: 'The UK Independence Party has stormed to victory in the European elections and unleashing a political whirlwind in Britain.',
       title: 'Farage throws down gauntlet as Ukip unleashes whirlwind',
       article: 'http://www.independent.ie/world-news/europe/farage-throws-down-gauntlet-as-ukip-unleashes-whirlwind-30306752.html',
       cpsid: 'irish_independant_7ddba5bc3c79dca7f45714c782c73e61625d8b85',
       published: '2014-05-27T01:30:00Z' } ]
  thumbnail: 'http://upload.wikimedia.org/wikipedia/commons/thumb/8/80/Official-photo-cameron.png/200px-Official-photo-cameron.png',
  name: 'David Cameron',
  type: 
   [ 'http://dbpedia.org/ontology/OfficeHolder',
     'http://dbpedia.org/ontology/Person',
     'http://dbpedia.org/ontology/Agent' ],
  uri: 'http://dbpedia.org/resource/David_Cameron' }
```

### getRelatedConcepts()

You can fetch concepts that - by being linked through news articles - are linked to a given concept. For example, to get concepts related to "Ukraine":

``` javascript
var apiKey = '1234567890ABCDEF';
var newsQuery = require('newsquery')(apiKey);
newsQuery.getRelatedConcepts("http://dbpedia.org/resource/Ukraine", 5)
.then(function(concepts) {
    console.log(concepts);
});
```

The response from findConcepts() is an array of concepts, returned in order of how many co-occurences there are between the concepts.

i.e. how much times both concepts have been mentioned in the same article, tagged in one image, mentioned in a video, etc.

``` javascript
[ { name: 'Russia',
    uri: 'http://dbpedia.org/resource/Russia',
    occurrences: 9530,
    image: 'http://upload.wikimedia.org/wikipedia/commons/thumb/f/f3/Flag_of_Russia.svg/200px-Flag_of_Russia.svg.png' 
  }
]
````

### getArticlesByConcept()

Once you have the URI for a concept, you can use it to find news articles mentioning it. You can specify a single concept URI or an array of URIs.

``` javascript
var apiKey = '1234567890ABCDEF';
var newsQuery = require('newsquery')(apiKey);
newsQuery.getArticlesByConcept(["http://dbpedia.org/resource/David_Cameron"], 5)
.then(function(articles) {
    console.log(articles);
});
```

Example response from getArticlesByConcept():

``` javascript
[ { id: 'the_mirror_ee58bebcd283cea0f654207ea051b8b2027263dd',
  source: 'http://www.bbc.co.uk/ontologies/bbc/TheMirror',
  url: 'http://www.mirror.co.uk/news/uk-news/conservatives-liberal-democrats-face-local-3590400',
  dateCreated: '2014-05-22T19:18:37Z',
  title: 'Conservatives and Liberal Democrats face local and European election catastrophe',
  description: 'Labour are battling to stop UKIP pulling off an unprecedented victory in the Euro polls - but are confident of winning the battle for council seats',
  image: 'http://i4.mirror.co.uk/incoming/article3586292.ece/alternates/s615/David-Cameron-and-his-wife-Samantha.jpg',
  concepts: 
   [ { name: 'London',
       uri: 'http://dbpedia.org/resource/London',
       type: 'Place',
       image: 'http://upload.wikimedia.org/wikipedia/commons/thumb/8/83/London_collage.jpg/200px-London_collage.jpg',
       lat: '51.507222222222225',
       lon: '-0.1275' },
     { name: 'Cambridge',
       uri: 'http://dbpedia.org/resource/Cambridge',
       type: 'Place',
       image: 'http://upload.wikimedia.org/wikipedia/commons/thumb/b/b4/KingsCollegeChapelWest.jpg/200px-KingsCollegeChapelWest.jpg',
       lat: '52.205',
       lon: '0.119' },
     { name: 'Brussels',
       uri: 'http://dbpedia.org/resource/Brussels',
       type: 'Place',
       image: 'http://upload.wikimedia.org/wikipedia/commons/thumb/c/c6/TE-Collage_Brussels.png/200px-TE-Collage_Brussels.png',
       lat: '50.85',
       lon: '4.35' },
     { name: 'David Cameron',
       uri: 'http://dbpedia.org/resource/David_Cameron',
       type: 'Person',
       image: 'http://upload.wikimedia.org/wikipedia/commons/thumb/8/80/Official-photo-cameron.png/200px-Official-photo-cameron.png' },
     { name: 'Liberal Democrats',
       uri: 'http://dbpedia.org/resource/Liberal_Democrats',
       type: 'Organisation',
       image: 'http://upload.wikimedia.org/wikipedia/commons/thumb/1/1f/Liberal_Democrats_Logo.svg/200px-Liberal_Democrats_Logo.svg.png' },
     { name: 'Ed Miliband',
       uri: 'http://dbpedia.org/resource/Ed_Miliband',
       type: 'Person',
       image: 'http://upload.wikimedia.org/wikipedia/commons/thumb/9/9c/Ed_Miliband_on_August_27,_2010_cropped-an_less_red-2.jpg/200px-Ed_Miliband_on_August_27,_2010_cropped-an_less_red-2.jpg' },
     { name: 'Nick Clegg',
       uri: 'http://dbpedia.org/resource/Nick_Clegg',
       type: 'Person',
       image: 'http://upload.wikimedia.org/wikipedia/commons/thumb/6/68/NickClegg_worldeconomic.jpg/200px-NickClegg_worldeconomic.jpg' },
     { name: 'Israeli Labor Party',
       uri: 'http://dbpedia.org/resource/Israeli_Labor_Party',
       type: 'Organisation',
       image: 'http://upload.wikimedia.org/wikipedia/commons/thumb/7/7b/Labor_(Israel)_logo.png/200px-Labor_(Israel)_logo.png' },
     { name: 'Malcolm Bruce',
       uri: 'http://dbpedia.org/resource/Malcolm_Bruce',
       type: 'Person',
       image: 'http://upload.wikimedia.org/wikipedia/commons/thumb/2/27/Malcolm_Bruce,_September_2009_cropped.jpg/200px-Malcolm_Bruce,_September_2009_cropped.jpg' },
     { name: 'Nigel Farage',
       uri: 'http://dbpedia.org/resource/Nigel_Farage',
       type: 'Person',
       image: 'http://upload.wikimedia.org/wikipedia/commons/thumb/4/4c/Nigel_Farage.jpg/200px-Nigel_Farage.jpg' },
     { name: 'Trafford',
       uri: 'http://dbpedia.org/resource/Trafford',
       type: 'Place',
       image: 'http://upload.wikimedia.org/wikipedia/commons/thumb/b/b4/Trafford-town-hall3.jpg/200px-Trafford-town-hall3.jpg',
       lat: '53.44611111111111',
       lon: '-2.3080555555555557' },
     { name: 'UK Independence Party',
       uri: 'http://dbpedia.org/resource/UK_Independence_Party',
       type: 'Organisation',
       image: 'http://upload.wikimedia.org/wikipedia/commons/thumb/d/d2/UKIP_logo.png/200px-UKIP_logo.png' },
     { name: 'Parliament of the United Kingdom',
       uri: 'http://dbpedia.org/resource/Parliament_of_the_United_Kingdom',
       type: 'Organisation',
       image: 'http://upload.wikimedia.org/wikipedia/commons/thumb/a/af/Crowned_Portcullis.svg/200px-Crowned_Portcullis.svg.png',
       lat: '51.49930555555556',
       lon: '-0.12475' },
     { name: 'Conservative Party (UK)',
       uri: 'http://dbpedia.org/resource/Conservative_Party_(UK)',
       type: 'Organisation',
       image: 'http://upload.wikimedia.org/wikipedia/commons/thumb/b/b6/Conservative_logo_2006.svg/200px-Conservative_logo_2006.svg.png' },
     { name: 'Coalition (Australia)',
       uri: 'http://dbpedia.org/resource/Coalition_(Australia)',
       type: 'Organisation',
       image: 'http://upload.wikimedia.org/wikipedia/commons/thumb/1/1c/LiberalpartyofausCROP.png/200px-LiberalpartyofausCROP.png' },
     { name: 'Labour Party (UK)',
       uri: 'http://dbpedia.org/resource/Labour_Party_(UK)',
       type: 'Organisation',
       image: 'http://upload.wikimedia.org/wikipedia/commons/thumb/0/05/Logo_Labour_Party.svg/200px-Logo_Labour_Party.svg.png' } ]
    }
]
````