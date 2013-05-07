#! /usr/bin/python

import os
import commands
import re


###################################################################
# configuration ends here - don't modify any line below this one
###################################################################

env = Environment()

###################################################################
# fix environment vars (not imported by default)
###################################################################


PATH=''
if 'PATH' in os.environ:
	PATH=os.environ['PATH']

LD_LIBRARY_PATH=''
if 'LD_LIBRARY_PATH' in os.environ:
	LD_LIBRARY_PATH = os.environ['LD_LIBRARY_PATH']

env = Environment(ENV = {'PATH' : PATH, 'LD_LIBRARY_PATH' : LD_LIBRARY_PATH})

###################################################################
# Command line options
###################################################################

#
# build directory
#
AddOption(	'--buildname',
		dest='buildname',
		type='string',
		nargs=1,
		action='store',
		help='build name (and output directory), default: \'./build\'')

env['buildname'] = GetOption('buildname')
if (env['buildname'] == None):
	env['buildname'] = 'build'



#
# compiler (gnu/intel)
#
AddOption(	'--compiler',
		dest='compiler',
		type='string',
		nargs=1,
		action='store',
		help='specify compiler to use (gnu/intel), default: gnu')

env['compiler'] = GetOption('compiler')

if (env['compiler'] == None or (env['compiler'] not in ['gnu', 'intel', 'open64'])):
	env['compiler'] = 'gnu'





#
# compile mode (debug/release)
#
AddOption(	'--mode',
		dest='mode',
		type='string',
		nargs=1,
		action='store',
		help='specify release or debug mode (release/debug), default: release')

env['mode'] = GetOption('mode')

if (env['mode'] == None or (env['mode'] not in ['release', 'debug'])):
	env['mode'] = 'release'



###################################################################
# SETUP COMPILER AND LINK OPTIONS
###################################################################


# add nvidia lib path when running on atsccs* workstation
hostname = commands.getoutput('uname -n')
if re.match("atsccs.*", hostname):
	env.Append(LIBPATH=['/usr/lib/nvidia-current/'])

env.Append(LIBPATH=['./libs/'])

#env.Append(LIBPATH=[os.environ['HOME']+'/local/lib'])
#env.Append(LIBS=['GL'])
if env ['PLATFORM'] == "darwin":
        env['FRAMEWORKS'] = ['OpenCL']
else:
        env.Append(LIBS=['OpenCL'])

# linking to the unit test library
env.Append(LIBS=['UnitTest++'])

#
# SDL
#
#reqversion = [2,0,0]
#sdlversion = commands.getoutput('sdl2-config --version').split('.')

#for i in range(0, 3):
#	if (int(sdlversion[i]) > int(reqversion[i])):
#		break;
#	if (int(sdlversion[i]) < int(reqversion[i])):
#		print 'libSDL Version 2.0.0 necessary.'
#		Exit(1)

#env.ParseConfig("sdl2-config --cflags --libs")
#env.ParseConfig("pkg-config freetype2 --cflags --libs")

#
# FREETYPE2
#
env.ParseConfig("pkg-config freetype2 --cflags --libs")

#
# SDL IMAGE
#
#if env ['PLATFORM'] == "darwin":
#       env['FRAMEWORKS'] = ['SDL2_image']
#else:
#        env.Append(LIBS=['SDL2_image'])

#
# xml
#
env.ParseConfig("pkg-config libxml-2.0 --cflags --libs")

if env['compiler'] == 'gnu':
#	env.Append(LINKFLAGS=' -static-libgcc')

	# eclipse specific flag
	env.Append(CXXFLAGS=' -fmessage-length=0')

	# be pedantic to avoid stupid programming errors
	env.Append(CXXFLAGS=' -pedantic')



if env['compiler'] == 'intel':
	# eclipse specific flag
	env.Append(CXXFLAGS=' -fmessage-length=0')

	# activate intel C++ compiler
	env.Replace(CXX = 'icpc')



if env['mode'] == 'debug':
	env.Append(CXXFLAGS=' -DDEBUG=1')

	if env['compiler'] == 'gnu':
		env.Append(CXXFLAGS=' -O0 -g3 -Wall')

	elif env['compiler'] == 'intel':
		env.Append(CXXFLAGS=' -O0 -g3')
#		env.Append(CXXFLAGS=' -traceback')

elif env['mode'] == 'release':
	env.Append(CXXFLAGS=' -DNDEBUG=1')

	if env['compiler'] == 'gnu':
		env.Append(CXXFLAGS=' -O3 -g -mtune=native')

	elif env['compiler'] == 'intel':
		env.Append(CXXFLAGS=' -xHOST -O3 -g -fast -fno-alias')

else:
	print 'ERROR: mode'
	Exit(1)



###################################################################
# DEPENDENCIES
###################################################################

# also include the 'src' directory to search for dependencies
#env.Append(CPPPATH = ['.', 'src/'])

# include the third party softwares like unit testing, ...
includePaths = [
        '#/include/UnitTest++',
        '.',
        'src/'
]
env.Append(CPPPATH=includePaths)

######################
# INCLUDE PATH
######################

# ugly hack
for i in ['src/', 'src/include/']:
	env.Append(CXXFLAGS = ' -I'+os.environ['PWD']+'/'+i)


######################
# setup PROGRAM NAME base on parameters
######################
program_name = 'lbm_opencl_dc'

# compiler
program_name += '_'+env['compiler']

# mode
program_name += '_'+env['mode']

######################
# get source code files
######################

env.src_files = []

Export('env')
SConscript('tests/SConscript', variant_dir='build/build_tests_'+program_name, duplicate=0)
SConscript('src/SConscript', variant_dir='build/build_'+program_name, duplicate=0)
Import('env')

print
print 'Building program "'+program_name+'"'
print

env.Program('build/'+program_name, env.src_files)

#Exit(0)