# PCDM in Sufia with Fedora 4
This is an attempt to map the example described in [LDP-PCDM-F4 In Action](https://wiki.duraspace.org/display/FEDORA4x/LDP-PCDM-F4+In+Action) to the classes that we will have in Sufia. 

These are the two base diagrams that I am using for this mapping:
* [Sufia PCDM](https://docs.google.com/drawings/d/1uTbg0FPQDoa2zN6p1I37m-M3CFnlx85Mp9CEyRw-rf4/edit)
* [Fedora PCDM](https://wiki.duraspace.org/download/attachments/68067286/ldp-pcdm-f4-dirs.png?version=2&modificationDate=1428639700927&api=v2)

LDP-PCDM-F4 in Action uses the example of a work "Raven" with a few files, that is associated to the "Poe" collection. For the purposes of the mapping to Sufia I am using "genfiles" instead of "pages". I am also using two different kind of files (cover.jpg and raven.pdf) to show that different kind of derivatives will be saved for each file depending on their type.


*BEGIN ELR MODIFICATION*

Here is a slight adjustment based on the assumptions from which I have been working.

Changes:
* addition of proxies for GenericFiles
* /works/raven/members is an IC
* insertion of Hydra::Works between Sufia specific class and Hydra::PCDM.  Unless Sufia needs to extend the definitions of GenericFile and GenericWork even more, Sufia should use Hydra::Works classes.

```
LDP  Fedora URI                                       Sufia class         Hydra::Works class         Hydra::PCDM class      
---  -----------------------------------------------  ------------------  -------------------------  -----------------   
BC   /works                
BC   /works/raven                                     Sufia::Work         Hydra::Works::GenericWork  Hydra::PCDM::Object    
IC   /works/raven/members
RS   /works/raven/members/coverjpgProxy
RS   /works/raven/members/ravenpdfProxy

DC   /works/raven/genfiles                            [1]        
                   
BC   /works/raven/genfiles/cover.jpg                  Sufia::GenericFile  Hydra::Works::GenericFile  Hydra::PCDM::Object    
DC   /works/raven/genfiles/cover.jpg/files            [1]                
NR   /works/raven/genfiles/cover.jpg/files/content    Sufia::File         Hydra::Works::File         Hydra::PCDM::File      
NR   /works/raven/genfiles/cover.jpg/files/thumbnail  Sufia::File         Hydra::Works::File         Hydra::PCDM::File      
                
BC   /works/raven/genfiles/raven.pdf                  *Sufia::GenericFile*  Hydra::Works::GenericFile  Hydra::PCDM::Object  
DC   /works/raven/genfiles/raven.pdf/files            [1]                
NR   /works/raven/genfiles/raven.pdf/files/content    Sufia::File         Hydra::Works::File         Hydra::PCDM::File      
NR   /works/raven/genfiles/raven.pdf/files/thumbnail  Sufia::File         Hydra::Works::File         Hydra::PCDM::File      
NR   /works/raven/genfiles/raven.pdf/files/text       Sufia::File         Hydra::Works::File         Hydra::PCDM::File      

BC   /collections
BC   /collections/poe                                 Sufia:Collections   Hydra::Works::Collection   Hdyra::PCDM::Collection
IC   /collections/poe/members
RS   /collections/poe/members/ravenProxy
```

Fetching `/works/raven` will return 

    </works/raven> pcdm:hasMember </works/raven/genfiles/cover.jpg> .
    </works/raven> pcdm:hasMember </works/raven/genfiles/raven.pdf> .

Fetching `/works/raven/genfiles/cover.jpg` will return 

    </works/raven/genfiles/cover.jpg> pcdm:hasFile </works/raven/genfiles/cover.jpg/files/content> .
    </works/raven/genfiles/cover.jpg> pcdm:hasFile </works/raven/genfiles/cover.jpg/files/thumbnail> .
    
Fetching `/collections/poe` will return

    </collections/poe> pcdm:hasMember </works/raven> .
    

**Question:** Will the Direct and Indirect containers have a mapping to Ruby classes or will their existence be handled behind the scenes by Active Fedora?

I expect that activefedora-aggregation's aggregates code to handle...
* The creation of direct and indirect containers as defined by Fedora's implementation of ORE:Aggregations.

I expect Hydra::PCDM to define...
* predicates for aggregation that subclass from ORE.aggregates predicate (e.g. hasMember is a subclass of ORE.aggregates)

There was discussion of being able to configure the directory structure to use in Fedora (e.g. /works/raven from the example here).  
I haven't seen any code yet that leads me to believe this configuration is anything but a possibility at some point in time.
Justin or others care to comment?

----
Comments:

I have heard stated...
* a File can live in one-and-only-one GenericFile
* a GenericFile can live in one-and-only-one GenericWork (debated)
 * Converse: a GenericFile can live in multiple GenericWorks (debated)
* a GenericFile can not be a member of a GenericFile (debated)
 * Converse: a GenericFile can be a member of a GenericFile (debated)

Putting GenericFiles under GenericWorks seems to lock us into one of these perspectives.  I am ok with going with restrictions on 
GenericFile for the sprint, but I don't want to paint ourselves into a corner that limits our choices in the future when we
re-examine our assumptions.


Here is a more extensive adjustment which does not make assumptions about GenericFiles in the future.

    LDP  Fedora URI                                       Sufia class         Hydra::Works class         Hydra::PCDM class      
    ---  -----------------------------------------------  ------------------  -------------------------  -----------------   
    BC   /works                
    BC   /works/raven                                     Sufia::Work         Hydra::Works::GenericWork  Hydra::PCDM::Object    
    IC   /works/raven/members
    RS   /works/raven/members/coverjpgProxy
    RS   /works/raven/members/ravenpdfProxy

    DC   /genfiles                                                 
    BC   /genfiles/cover.jpg                              Sufia::GenericFile  Hydra::Works::GenericFile  Hydra::PCDM::Object    
    DC   /genfiles/cover.jpg/files                        [1]                
    NR   /genfiles/cover.jpg/files/content                Sufia::File         Hydra::Works::File         Hydra::PCDM::File      
    NR   /genfiles/cover.jpg/files/thumbnail              Sufia::File         Hydra::Works::File         Hydra::PCDM::File      
                    
    BC   /genfiles/raven.pdf                              *Sufia::GenericFile*  Hydra::Works::GenericFile  Hydra::PCDM::Object  
    DC   /genfiles/raven.pdf/files                        [1]                
    NR   /genfiles/raven.pdf/files/content                Sufia::File         Hydra::Works::File         Hydra::PCDM::File      
    NR   /genfiles/raven.pdf/files/thumbnail              Sufia::File         Hydra::Works::File         Hydra::PCDM::File      
    NR   /genfiles/raven.pdf/files/text                   Sufia::File         Hydra::Works::File         Hydra::PCDM::File      
    
    BC   /collections
    BC   /collections/poe                                 Sufia:Collections   Hydra::Works::Collection   Hdyra::PCDM::Collection
    IC   /collections/poe/members
    RS   /collections/poe/members/ravenProxy


Fetching `/works/raven` will return 

    </works/raven> pcdm:hasMember </genfiles/cover.jpg> .
    </works/raven> pcdm:hasMember </genfiles/raven.pdf> .

Fetching `/genfiles/cover.jpg` will return 

    </genfiles/cover.jpg> pcdm:hasFile </genfiles/cover.jpg/files/content> .
    </genfiles/cover.jpg> pcdm:hasFile </genfiles/cover.jpg/files/thumbnail> .
    
Fetching `/collections/poe` will return

    </collections/poe> pcdm:hasMember </works/raven> .
    
*END ELR MODIFICATION*

Key for the LDP column (remember that all containers are also RDF sources)
* BC: Basic Container
* DC: Direct Container
* IC: Indirect Container
* NR: Non-RDF Source
* RS: RDF Source

You should also take a look at Adam Wead's [examples](https://github.com/awead/sufia-pcdm) on this topic.


# Run the sample in Fedora 4 
Run `work_raven.sh` to create these objects in Fedora 4.
