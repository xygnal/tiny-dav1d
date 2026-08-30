# Copyright © 2020, VideoLAN and dav1s authors
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#
# 1. Redistributions of source code must retain the above copyright notice, this
#    list of conditions and the following disclaimer.
#
# 2. Redistributions in binary form must reproduce the above copyright notice,
#    this list of conditions and the following disclaimer in the documentation
#    and/or other materials provided with the distribution.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
# ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
# WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
# DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR
# ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
# (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
# LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
# ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
# SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

#
# Build definition for the dav1s fuzzing binaries
#

if fuzzing_engine == 'none' and not have_fseeko
    subdir_done()
endif

dav1s_fuzzer_sources =  files('dav1s_fuzzer.c')
fuzzer_ldflags = []
fuzzer_link_lang = {}

if get_option('fuzzer_ldflags') != ''
    fuzzer_ldflags += [get_option('fuzzer_ldflags')]
endif

if fuzzing_engine == 'none'
    dav1s_fuzzer_sources += files('main.c')
elif fuzzing_engine == 'libfuzzer'
    fuzzer_ldflags += ['-fsanitize=fuzzer']
elif fuzzing_engine == 'oss-fuzz'
    # libFuzzingEngine needs c++
    add_languages('cpp')
    fuzzer_link_lang = {'link_language': 'cpp'}
endif

dav1s_fuzzer = executable('dav1s_fuzzer',
    dav1s_fuzzer_sources,
    include_directories: dav1s_inc_dirs,
    link_args: fuzzer_ldflags,
    link_with : libdav1s,
    build_by_default: true,
    dependencies : [thread_dependency],
    kwargs: fuzzer_link_lang
    )

dav1s_fuzzer_mt = executable('dav1s_fuzzer_mt',
    dav1s_fuzzer_sources,
    include_directories: dav1s_inc_dirs,
    c_args: ['-DDAV1S_MT_FUZZING'],
    link_args: fuzzer_ldflags,
    link_with : libdav1s,
    build_by_default: true,
    dependencies : [thread_dependency],
    kwargs: fuzzer_link_lang
    )

objcopy = find_program('objcopy',
                       required: false)
if (objcopy.found() and
    not get_option('b_lto') and
    get_option('default_library') == 'static' and
    cc.has_function('posix_memalign', prefix : '#include <stdlib.h>', args : test_args))

    libdav1s_af = custom_target('libdav1s_af',
                                input: libdav1s,
                                output: 'libdav1s_af.a',
                                depends: libdav1s,
                                command: [objcopy,
                                          '--redefine-sym', 'malloc=__wrap_malloc',
                                          '--redefine-sym', 'posix_memalign=__wrap_posix_memalign',
                                          '--redefine-sym', 'pthread_create=__wrap_pthread_create',
                                          '--redefine-sym', 'pthread_cond_init=__wrap_pthread_cond_init',
                                          '--redefine-sym', 'pthread_mutex_init=__wrap_pthread_mutex_init',
                                          '@INPUT@', '@OUTPUT@'])

    dav1s_fuzzer_mem = executable('dav1s_fuzzer_mem',
        dav1s_fuzzer_sources + ['alloc_fail.c'],
        include_directories: dav1s_inc_dirs,
        c_args: ['-DDAV1S_ALLOC_FAIL'],
        link_args: fuzzer_ldflags + [join_paths(libdav1s_af.full_path())],
        link_depends: libdav1s_af,
        build_by_default: false,
        dependencies : [thread_dependency],
        kwargs: fuzzer_link_lang
        )
endif
