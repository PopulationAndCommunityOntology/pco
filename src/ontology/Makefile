OBO=http://purl.obolibrary.org/obo
ONT=pco
BASE=$(OBO)/$(ONT)
SRC=$(ONT)-edit.owl
RELEASEDIR=../..
ROBOT= robot
OWLTOOLS= owltools
# USECAT= --use-catalog

all: all_imports merge
test: all
prepare_release: all
	cp $(ONT).owl $(RELEASEDIR) &&\
	mkdir -p $(RELEASEDIR)/imports &&\
	cp imports/*{owl,obo} $(RELEASEDIR)/imports &&\
	git add $(RELEASEDIR)/imports/*{obo,owl} &&\
	(test -f subsets/*owl && cp subsets/*{owl,obo} $(RELEASEDIR)/subsets && git add $(RELEASEDIR)/subsets/* || echo "no subsets") &&\
	echo "Release files are now in $(RELEASEDIR) - now you should commit, push and make a release on github"

$(ONT).owl: $(SRC)
	$(ROBOT)  reason -i $< -r ELK relax reduce -r ELK annotate -V $(BASE)/releases/`date +%Y-%m-%d`/$(ONT).owl -o $@


IMPORTS = ro pato envo go ncbitaxon bfo
IMPORTS_OWL = $(patsubst %, imports/%_import.owl,$(IMPORTS))
# Make this target to regenerate ALL
all_imports: $(IMPORTS_OWL)

# Use ROBOT, driven entirely by terms lists NOT from source ontology
imports/%_import.owl: mirror/%.owl imports/%_terms.txt
	$(ROBOT) extract -i $< -T imports/$*_terms.txt --method BOT -O $(BASE)/$@ -o $@
.PRECIOUS: imports/%_import.owl

# clone remote ontology locally, perfoming some excision of relations and annotations
mirror/%.owl: $(SRC)
	$(OWLTOOLS) $(OBO)/$*.owl --remove-annotation-assertions -l -s -d --remove-dangling-annotations  -o $@
.PRECIOUS: mirror/%.owl

release: merge
	cp $(ONT).owl $(RELEASEDIR) && cp imports/* $(RELEASEDIR)/imports

### only the imports you want to merge are in the imports directory and match the wilcard pattern below
IMPORTS_MERGE = $(patsubst %_import.owl, --input %_import.owl, $(wildcard imports/*_import.owl)) 
merge: $(ONT).owl
	$(ROBOT) merge --input $< $(IMPORTS_MERGE) --output $<
