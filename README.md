# Publisher API

The Yewno Semantic API enables users to retrieve comprehensive information on topics, subtopics and concepts from any English language item in the relevant portfolio that they have access to. This information has a variety of uses and can be displayed both internally and externally to end users. Overall, the Yewno Sematic API is composed of six related APIs: Concept Search, Concept Metadata, Document Search, Document Metadata, Document Snippets and Topics outlined below. 

The base url for all endpoints is `https://apis.yewno.com/public`.  All requests must be made over HTTPS as HTTP is not supported.  

## Assumptions
- Only support `json` content bodies.
- Every API request must contain a JWT token.
- *This is a draft spec, specific response fields may change*

## Glossary
__Concept__: An abstract idea that may refer to one or more terms. e.g. _Brexit_, _United Kingdom European Union membership referendum, 2016_  
__Document__: A piece of content; book, chapter, article, etc.  
__Topic__: A high level concept classification that may be further divided into specific subtopics. eg _'cat'_  
__Subtopic__: A more granular concept classification beneath a topic, eg _'Ragdoll'_ (cat breed).  
__yid__: a globally unique id assigned to each document resource. eg `1f84027593d21e4f3a01ef676bebf4db`  
__cid__: a globally unique id assigned to each concept eg `1a267fae7720d4bfc71abd6494611cf5`

## Authentication

All requests must be accompanied by a [jwt](https://jwt.io/introduction/) obtained by logging in with a registered account. 

### Registration

Visit `https://auth.yewno.com/register` to register for an account.

### Obtaining a JWT

Once registered, you may acquire a jwt from the authentication gateway.  All jwts expire 24hrs after they are issued.  You may also request a refresh token with the optional `permanent` field.  This refresh token is tied to your current ip address and will not renew your session from another address.

```
POST https://auth.yewno.com/api/login

{
  "email": "youremail@yourdomain.com",
  "password": "your password",
  "permanent": true
}
```

This provided token must be present in each request as a standard formatted authorization bearer token.  In other words, all API requests must include the following authorization header: 

``` 
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### Associating with an institute

To use the API, your account must be associated with a institute.  Contact Yewno for a token and POST it to: 

```
POST https://auth.yewno.com/api/account/coupon

{
  "code": "YOURCODEHERE"
}
```

Once you have applied this request, you will need to obtain a new JWT that contains the newly acquired association.  You can verify access by decoding the newly created JWT and noting additional entries in the `roles` field.

### Renew Session

To renew the session, post the refresh token to `https://auth.yewno.com/auth/refresh`: 

```
POST /auth/refresh
{
  "refresh": "24e19203-57cd-4eec-ad56-5ad35b1afdaf"
}
```


## Concept Search

Concept search API returns most pertinent concept(s) matched to a query string. It handles misspelled or partial data, as well as synonyms. For every concept returned, the API provides its id within Yewno ecosystem (cid), together with concept title and short definition. Additional metadata for a given yid can be obtained through the Concept Metadata API (see below). We limit the number of returned concepts to 10. 

```
GET /concept/search?q=<string>
```

| param | type |
|---|---|
| q  | string  |

<details>

```json
{
  "data": [
    {
      "definitions": [
        {
	        "score": 0.9,
	        "resource": "wikipedia:25202",
	        "year": "2017",
	        "description": "Quantum mechanics (QM; also known as quantum physics or quantum theory), including quantum field theory, is a branch of physics which is the fundamental theory of nature at the small scales and energy levels of atoms and subatomic particles. Classical physics (the physics existing before quantum mechanics) derives from quantum mechanics as an approximation valid only at large (macroscopic) scales. Quantum mechanics differs from classical physics in that: energy, momentum and other quantities are often restricted to discrete values (quantization), objects have characteristics of both particles and waves (i.e. wave-particle duality), and there are limits to the precision with which quantities can be known (uncertainty principle). (Landau & Lifshitz, 1977) Quantum mechanics gradually arose from Max Planck's solution in 1900 to the black-body radiation problem (reported 1859) and Albert Einstein's 1905 paper which offered a quantum-based theory to explain the photoelectric effect (reported 1887).",
	        "label": "Wikipedia",
	        "source": "wikipedia",
	        "id": "b97718aa9aed76997c404f5965ac629c",
	        "title": "Quantum mechanics",
	        "url": "https://en.wikipedia.org/wiki/Quantum_mechanics",
	        "sourceLabel": "wikipedia"
        }
      ],
      "cId": "1a267fae7720d4bfc71abd6494611cf5",
      "topics": [
        {
          "topic": "21",
          "score": 0.53311855,
          "children": [
            {
              "topic": "21_22",
              "score": 0.85977834,
              "id": "21_22",
              "title": "Quantum Theory"
            }
          ],
          "id": "21",
          "title": "Physics"
        },
        {
          "topic": "23",
          "score": 0.16696094,
          "children": [
            {
              "topic": "23_5",
              "score": 0.7479905,
              "id": "23_5",
              "title": "Cognitive Science"
            },
            {
              "topic": "23_6",
              "score": 0.20375429,
              "id": "23_6",
              "title": "Connectionism"
            }
          ],
          "id": "23",
          "title": "Psychology"
        }
      ],
      "title": "Quantum mechanics",
      "imageUrl": "https://static.yewno.com/assets/thumbnails/concepts/1a267fae7720d4bfc71abd6494611cf5@2x.jpg"
    },
    /* ... */
  ]
}
```

</details>

## Concept Metadata

Concept metadata returns, for each concept id (cid), additional attributes obtained via Yewno advanced AI processes. These include the topics/subtopics hierarchy together with a percentage for each, definitions and a list of related concept CIDs (up to 100) extracted from multiple sources (depending on the portfolio a particular user has access to).

| param | type |
|---|---|
| cid | string |
| related* | integer | 

\* optional

```
GET /concept/:cid?related=<int>
```

<details>

```json
{
  "status": 200,
  "uuid": "0deee814-74da-11e8-a038-0242ac110003",
  "cache_disabled": false,
  "meta": {
    "context": []
  },
  "message": "OK",
  "data": {
    "nodes": [
      {
        "cId": "40dec4929a8ad3cda7a90e53394eb03b",
        "topics": [
          {
            "topic": "28",
            "probability": 0.33009732,
            "subtopics": [
              {
                "topic": "28_54",
                "probability": 0.9117648
              }
            ]
          }
        ],
        "title": "ABW (TV station)",
        "sourceName": "wikipedia",
        "definitions": [
          {
            "resource": "wikipedia:3165227",
            "description": "ABW is the Australian Broadcasting Corporation's television station in Perth, Western Australia. The station began broadcasting on 7 May 1960 from studios on Adelaide Terrace in downtown Perth and its transmitter at Bickley. The station is relayed throughout the state by a number of transmitters as well as on the Optus Aurora free-to-view satellite television platform. In 2005 the station moved to a new digital broadcast centre in East Perth.",
            "title": "ABW (TV station)",
            "url": "https://en.wikipedia.org/wiki/ABW_%28TV_station%29",
            "label": "Wikipedia",
            "source": "wikipedia",
            "score": 0.9,
            "year": "2017",
            "sourceLabel": "Wikipedia",
            "id": "35533cfcc8ff9e787dfe78b62bfe358f"
          }
        ],
        "imageUrls": [
          "https://static.yewno.com/assets/thumbnails/concepts/40dec4929a8ad3cda7a90e53394eb03b@2x.jpg",
          "https://static.yewno.com/assets/thumbnails/topics_36/28@2x.jpg"
        ]
      },
    ],
    "edges": [
      {
        "source": "fe451a8fbf229b87a92fed8cb9289143",
        "target": "8ec2d886c4ab214daff14997506570ad",
        "similarity": 0.9928904233742654
      },
      /* ... */,
    ]
  }
}
```

</details>

## Document Search

Document search API returns most relevant document(s) matched to either a concept or a topic/subtopic. For every document returned, the API provides its id within Yewno ecosystem (yid), together with the document title, provider and date of publication.  Additional metadata for a given yid can be obtained through the Document Metadata API (see below).

| param | |
|---|---|
| q  | string  |
| stype  | ["topic", "concept"]  |

<details>

```
GET document/search?q=<string>&stype=<string> 
```
```json
{
  "data": [
    {
      "yId": "a53a456a1443b9f2eba1591e35800734",
      "stype": "title",
      "title": "Newsgames",
      "sourceName": "MIT Press",
    },
    /* ... */
  ]
}
```

</details>

## Document Metadata

Document metadata returns, for each document id (yid), traditional publisher metadata as well as additional attributes obtained via Yewno advanced AI algorithms. These include the **topics/subtopics** hierarchy together with the **percentage score**, and a list of **extracted concepts** (default response 10, the requester can optionally specify the number of related concepts she/he wants (up to max 100) with an associated **pertinence score**. Traditional metadata includes title, author(s), date of publication, as well as additional metadata depending on the type of document.

```
GET /document/:yid
```

<details>

```json
{
  "data": {
    "isbn": "",
    "text": "Other costs not included by this term are costs of follow-up literature, miscellaneous advertising materials such as circulars, layouts, letterheads, and calling cards, and cost of advertising agency service. ume of response delivered by different publications, although as mentioned, no substantial difference in the quality, when a known piece of copy was used. In those cases where the same copy appeared in different magazines in the same month, the ratio of orders to inquiries8 was quite similar for most magazines. 3. Direct mail inquiries as producers of traceable sales from both old9 and new10 8 The percentage of inquiries which were converted to orders. 9 A buyer whose first order preceded his first inquiry from advertising in the period measured, yet who did inquire from an advertisement or an editorial article, and who bought again after his inquiry was received.",
    "topics": [
      {
        "topic": "11_1",
        "probability": 0.8888889
      },
      {
        "topic": "11_15",
        "probability": 0.11111111
      },
      {
        "topic": "12",
        "children": [
          {
            "topic": "12_15",
            "name": "Publishing",
            "probability": 0.8322142
          },
          {
            "topic": "12_2",
            "name": "Communication Studies",
            "probability": 0.14093931
          }
        ],
        "probability": 0.13186002,
        "name": "Language Arts"
      },
      /*...*/
    ],
    "chapterNumber": "",
    "publishedAt": "2006-04-25T23:52:59Z",
    "readtime": 26,
    "title": "Traceable Response as a Method of Evaluating Industrial Advertising: A Case Study",
    "type": "article",
    "public": false,
    "yId": "1f84027593d21e4f3a01ef676bebf4db",
    "journal_title": "Journal of Marketing",
    "volume": "12",
    "authors": [
      {
        "lastName": "Margolis",
        "firstName": "Charles"
      }
    ],
    "concepts": [
    	{
    		"cid": "1a267fae7720d4bfc71abd6494611cf5",
    		"pertinence": 
    	}
    ],
    "publication_date": "2006-04-25T23:52:59Z",
    "eissn": "",
    "chapter": "",
    "publisher": "JSTOR",
    "doi": "10.2307/1245359",
    "sourcesList": [
      {
        "url": "http://dx.doi.org/10.2307/1245359",
        "type": "DISTRIBUTOR",
        "label": "JSTOR"
      }
    ],
    "issue": "2",
    "issn": "00222429"
  },
  /* ... */
}
```

</details>

## Document Snippets

Snippets endpoints returns, for each concept id (cid), list of snippets (short portions of documents, up to 180 words) where that concept appears, a relevance score, together with essential metadata information. Additional metadata for a given yid can be obtained through the Document Metadata API (see above) for each document id (yid) returned. Snippet call is paginated (with 10 snippets per page), and we limit the number of returned pages to 100. 

| param | type |
|-------|------|
| cid   | string |
| yids | string (comma separated yids) |

```
GET /snippets/:cid?yids=<string>
```

<details>

```
  "snippets": [
    {
      "yId": "d771a5b0bfd40e85efa9b4359d5a5432",
      "score": 4.0841975,
      "snippetIndex": 71,
      "sId": "63fc26375ec2343623661e41d7c5887e",
      "cmid": "659462e857811336fb48232d96b9e519",
      "text": " (a) Sketch indicating the rotation of a shuttlecock moving with the cork ahead. Thin blue arrows indicate the drag force on each feather. (b) Rotation velocity R &OHgr; ?> as a function of the translation speed U for a plastic (blue dots) and a feathered shuttlecock (red squares). Shuttlecocks rotate at a velocity such that this torque is balanced by air resistance. The rotational velocity &OHgr; is measured as a function of the projectile speed U , as shown in figure 15 (b). The graph reveals a linear correlation between R &OHgr; ?> and U , and differences between plastic and feather rotational velocities.",
      "title": "The physics of badminton",
      "publisher": "IOP Publishing",
      "summary": "...ed shuttlecock (red squares). Shuttlecocks rotate at a velocity such that this torque is balanced by air resistance. The rotational velocity &OHgr; is measured as a function of the projectile speed U , as shown in figure 15 (b). The graph reveals a linear...",
      "publicationDate": "2015-06-01T00:00:00Z"
    }
  ],
```

</details>

## Topics

Return topic and subtopic ids that can be used in the document search endpoint.

```
GET /topics
```

<details>

```
{
  "data": [
    {
      "id": "22",
      "label": "Political Science",
      "subtopics": [
        {
          "id": "22_7",
          "label": "Health Policy"
        },
        /* ... */
      ]
    },
    /* ... */
  ]
}
```

</details>
