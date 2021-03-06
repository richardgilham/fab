# The best way to show how a test case should be built is to build it.
# While we don't have a build system we will use someone elses.
#
# This test assumes that the PSyclone code generation tool is available on the
# executino path. i.e. That the directory containing it appears in the PATH
# environment variable.
#
.SUFFIXES:
.SUFFIXES: .x90 .f90 .F90 .o .mod

# The compiler has to be ifort as we use the "-module" argument to redirect
# module file storage.
#
export FC = ifort

PSYCLONE = psyclone

ifeq "x$(shell command -v $(PSYCLONE))" "x"
$(error Could not find the PSyclone script on PATH)
endif

DIRECTORIES = kernels model

objects: kernels/my_kernel_mod.o algorithm_mod.o algorithm_mod_psy.o

%.o %.mod: %.f90
	@echo Compiling $<
	$(FC) -o $*.o -module $(dir $@) $(addprefix -I,$(DIRECTORIES)) -c $<

%_psy.f90 %.f90: %.x90 optimisation.py
	@echo Psyclone $<
	$(PSYCLONE) -oalg $*.f90 -opsy $*_psy.f90  -d kernels \
                    -s $(realpath optimisation.py) \
                    -api dynamo0.3 -l -nodm $<

algorithm_mod.f90 \
algorithm_mod_psy.f90: algorithm_mod.x90 \
                       kernels/my_kernel_mod.f90 optimisation.py
algorithm_mod.o \
algorithm_mod.mod: algorithm_mod.f90 algorithm_mod_psy.mod \
                   model/field_mod.mod kernels/my_kernel_mod.mod
algorithm_mod_psy.o \
algorithm_mod_psy.mod: algorithm_mod_psy.f90 \
                       model/field_mod.mod model/operator_mod.mod \
                       kernels/my_kernel_mod.mod
kernels/my_kernel_mod.o \
kernels/my_kernel_mod.mod: kernels/my_kernel_mod.f90 \
                           model/argument_mod.mod model/constants_mod.mod \
                           model/functionspace_mod.mod model/kernel_mod.mod \

model/field_mod.o \
model/field_mod.mod: model/field_mod.f90 model/functionspace_mod.mod

clean:
	-rm -r *.o *.mod kernels/*.o kernels/*.mod model/*.o model/*.mod
	-rm *.pyc algorithm_mod.f90 algorithm_mod_psy.f90
