# Prerequisite variables
SOURCEDIR := src
OUTPUTDIR := build

SOURCES := $(shell find $(SOURCEDIR) -type f -name '*.java')
CLASSES := $(SOURCES:$(SOURCEDIR)/%.java=$(OUTPUTDIR)/%.class)

TARGETS := $(shell find $(SOURCEDIR) -mindepth 1 -maxdepth 1 -type d)
TARGETS := $(filter-out $(OUTPUTDIR)/lwjgl.jar,$(TARGETS:$(SOURCEDIR)/%=$(OUTPUTDIR)/%.jar))

# Compile the launcher compatibility layer against 3.4.1 first.  The 3.3.3
# jars remain available for older Minecraft versions, while the first matching
# class is the 3.4.1 API (including org.lwjgl.util.spvc.Spvc).
LWJGL34_JARS := $(wildcard libs/lwjgl-3.4.1/*.jar)
LWJGL33_JARS := $(wildcard libs/lwjgl-3.3.3/*.jar)
OTHER_JARS := $(wildcard libs/caciocavallo/*.jar) $(wildcard libs/caciocavallo17/*.jar) $(wildcard libs/others/*.jar)
JCFLAGS := -cp "$(subst $() $(),:,$(wildcard $(SOURCEDIR)/*/):$(LWJGL34_JARS) $(LWJGL33_JARS) $(OTHER_JARS))" -XDignore.symbol.file

all: $(TARGETS) $(OUTPUTDIR)/lwjgl-3.3.3.jar $(OUTPUTDIR)/lwjgl-3.4.1.jar
	
# Sometimes the CI produces empty class file resulting in broken jar, so we check and abort here
check_empty_classes:
	@set -e
	@echo "Checking for empty class files"
	$(eval EMPTY_CLASSES := $(shell find $(OUTPUTDIR) -name '*.class' -size 0))
	@if [ -n "$(EMPTY_CLASSES)" ]; then \
		echo "Error: found empty class files in $@: $(EMPTY_CLASSES)"; \
		exit 1; \
	fi

# Rules for building java class
$(CLASSES): $(OUTPUTDIR)/%.class: $(SOURCEDIR)/%.java
	@set -e
	$(eval TARGET=$(word 2, $(subst /, ,$@)))
	@mkdir -p $(OUTPUTDIR)/classes
	@mkdir -p $(dir $@)
	@echo "Compiling $<"
	@$(BOOTJDK)/javac $(JCFLAGS) -d $(OUTPUTDIR)/classes $<
	@cp $(@:$(OUTPUTDIR)/$(TARGET)/%.class=$(OUTPUTDIR)/classes/%*.class) $(dir $@)/

$(OUTPUTDIR)/launcher.jar: $(CLASSES) check_empty_classes
	@set -e
	@echo "Creating $@"
	@$(BOOTJDK)/jar -cf $@ -C $(basename $@) .

$(OUTPUTDIR)/lwjgl-3.3.3.jar: $(CLASSES) check_empty_classes
	@set -e
	@echo "Creating $@"
	@rm -rf $(OUTPUTDIR)/lwjgl_lib_3.3.3
	@mkdir -p $(OUTPUTDIR)/lwjgl_lib_3.3.3
	@cd $(OUTPUTDIR)/lwjgl_lib_3.3.3 && find ../../libs/lwjgl-3.3.3 -name 'lwjgl*.jar' -exec $(BOOTJDK)/jar -xf {} \; && rm -rf META-INF
	@cp -R $(OUTPUTDIR)/lwjgl/* $(OUTPUTDIR)/lwjgl_lib_3.3.3/
	@$(BOOTJDK)/jar -cf $@ -C $(OUTPUTDIR)/lwjgl_lib_3.3.3 .

$(OUTPUTDIR)/lwjgl-3.4.1.jar: $(CLASSES) check_empty_classes
	@set -e
	@echo "Creating $@"
	@rm -rf $(OUTPUTDIR)/lwjgl_lib_3.4.1
	@mkdir -p $(OUTPUTDIR)/lwjgl_lib_3.4.1
	@cd $(OUTPUTDIR)/lwjgl_lib_3.4.1 && find ../../libs/lwjgl-3.4.1 -name 'lwjgl*.jar' -exec $(BOOTJDK)/jar -xf {} \; && rm -rf META-INF
	@cp -R $(OUTPUTDIR)/lwjgl/* $(OUTPUTDIR)/lwjgl_lib_3.4.1/
	@$(BOOTJDK)/jar -cf $@ -C $(OUTPUTDIR)/lwjgl_lib_3.4.1 .

$(OUTPUTDIR)/patchjna_agent.jar: $(CLASSES)
	@set -e
	@mv $(basename $@)/com/sun/jna/Platform.class $(basename $@)/com/sun/jna/Platform.class.patch
	@echo "Creating $@"
	@$(BOOTJDK)/jar -cfm $@ $(SOURCEDIR)/patchjna_agent.txt -C $(basename $@) .

clean:
	rm -rf $(OUTPUTDIR)

.SUFFIXES: .java
.PHONY: all clean
