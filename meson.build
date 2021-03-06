project('libmemory',
	['c', 'cpp'],
	default_options : ['c_std=c11','cpp_std=c++17'])

########################
# Identify Compiler(s) #
########################

# Identify compilers & get flags
subdir('build/compiler')

# compiler_args is defined in build/compiler/<target>/meson.build
# We are the standard library, don't complain about __ prefixes
add_project_arguments([compiler_args, '-Wno-reserved-id-macro'], language: ['c', 'cpp'])

subdir('build/ea-stdlib')

#########################
# Process Build Options #
#########################

libc_target_include_directories = []

if get_option('use-ea-libc') == true
	ea_libc_path = get_option('ea-libc-path')
		message('Compiling with Embedded Artistry libc')
	message('Looking for libc at location: ' + ea_libc_path +
		'.  If libc is not found, update the ea-libc-path configuration option.')
	# TODO: can we check for a file to confirm?

	subdir('build/architecture')
	message('Building libmemory for target architecture: ' + target_architecture)
	libc_target_include_directories = [
		include_directories(ea_libc_path + '/include', is_system: true),
		include_directories(ea_libc_path + '/arch/' + target_architecture + '/include', is_system: true)
	]

	message('Building libmemory for native architecture: ' + native_architecture)
	libc_native_include_directories = [
		include_directories(ea_libc_path + '/include', is_system: true),
		include_directories(ea_libc_path + '/arch/' + native_architecture + '/include', is_system: true)
	]
endif

if get_option('enable-werror') == true
	add_project_arguments('-Werror', language : 'c')
endif

if get_option('enable-pedantic-error') == true
	add_project_arguments('-pedantic-error', language : 'c')
endif

#######################
# Process source tree #
#######################

# Used for local builds - we want the warnings
libmemory_includes = include_directories('include', 'dependencies/lib')

# Used for external builds - treat the headers as system headers
libmemory_system_includes = include_directories('include', 'dependencies/lib', is_system: true)

# Library build definition
subdir('src/')

# Test build definition
subdir('test/')

############################
# Supporting Build Targets #
############################

run_target('docs',
	command: ['doxygen', 'docs/Doxyfile'])

run_target('complexity',
	command: ['tools/complexity_check.sh', '-w'])

run_target('complexity-list',
	command: ['tools/complexity_check.sh'])


cppcheck = find_program('cppcheck', required: false)

if cppcheck.found()

	run_target('cppcheck',
		command: ['cppcheck',
		# Options
		'--quiet', '--enable=style', '--force',
		# Include Directories
		'-I', meson.current_source_dir() + '/include',
		'-I', meson.current_source_dir() + '/dependencies/lib',
		# Source Directories
		'src', 'test']
	)

	run_target('cppcheck-xml',
		command: ['tools/cppcheck_xml.sh']
	)

endif

clangtidy = find_program('clang-tidy', required: false)

if clangtidy.found()
    run_target(
        'tidy',
        command: [
            'build/tooling/clang-tidy/clang-tidy.sh',
            # Checks
            '-*,cppcoreguidelines-*,modernize-*,performance-*,portability-*,readability-*,' +
            '-readability-inconsistent-declaration-parameter-name,' +
            # Header Filter
            '.*',
            #Finish with files
        ] + libmemory_lint_files)
endif
