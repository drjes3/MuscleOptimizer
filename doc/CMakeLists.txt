# Author Elena Ceseracciu, Dipartimento di Tecnica e Gestione dei Sistemi Industriali, Università di Padova

option (BUILD_SPHINX_MANUAL_HTML "Build HTML version of the manual using Sphinx" OFF)
option (BUILD_SPHINX_MANUAL_HTML_SINGLEPAGE "Build HTML single-page version of the manual using Sphinx" OFF)
option (BUILD_SPHINX_MANUAL_LATEX "Build LaTex version of the manual using Sphinx" OFF)

if(BUILD_SPHINX_MANUAL_HTML OR BUILD_SPHINX_MANUAL_LATEX)
   find_program(SPHINX_EXECUTABLE
                  NAMES sphinx-build
                  DOC "Sphinx Documentation Builder (sphinx-doc.org)"
                  )
    if (NOT SPHINX_EXECUTABLE)
        message(WARNING "Sphinx could not be found, disabling manual generation")
        set(BUILD_SPHINX_MANUAL_HTML OFF CACHE BOOL "Build HTML version of the manual using Sphinx" FORCE)
        set(BUILD_SPHINX_MANUAL_LATEX OFF CACHE BOOL "Build Latex version of the manual using Sphinx" FORCE)
        set(BUILD_SPHINX_MANUAL_HTML_SINGLEPAGE OFF CACHE BOOL"Build HTML single-page version of the manual using Sphinx" FORCE)
    else()

    set(conf_copyright "2015  Elena Ceseracciu, Luca Modenese")

    set(conf_docs "${CMAKE_CURRENT_SOURCE_DIR}/help")
    set(conf_path "${CMAKE_CURRENT_SOURCE_DIR}")
    set(conf_version "${MO_MAJOR_VERSION}.${MO_MINOR_VERSION}.${MO_PATCH_VERSION}")
    set(conf_release "${MO_MAJOR_VERSION}.${MO_MINOR_VERSION}.${MO_PATCH_VERSION}")

    option(SPHINX_LATEX_PRINTABLE "Add page number to internal references (including bibliography). Useful to generated printer-friendly PDF" OFF)
    if(SPHINX_LATEX_PRINTABLE)
        set(latex_show_pagerefs "True")
        set(latex_show_urls "'inline'")
    else()
        set(latex_show_pagerefs "False")
        set(latex_show_urls "'no'")
    endif()
    configure_file(conf.py.in conf.py @ONLY)

    set(doc_formats "")
    if(BUILD_SPHINX_MANUAL_HTML)
      list(APPEND doc_formats html)
    endif()
    if(BUILD_SPHINX_MANUAL_HTML_SINGLEPAGE)
      list(APPEND doc_formats singlehtml)
    endif()
    if(BUILD_SPHINX_MANUAL_LATEX)
        find_package(LATEX)
          if (NOT PDFLATEX_COMPILER)
            message(FATAL_ERROR "pdflatex not found!")
          endif()
          list(APPEND doc_formats latex)

      set(latex_extra_commands
        COMMAND ${CMAKE_COMMAND} -E chdir ${CMAKE_CURRENT_BINARY_DIR}/latex
                ${PDFLATEX_COMPILER} ${CMAKE_CURRENT_BINARY_DIR}/latex/MuscleOptimizerHelp.tex
      )
    endif()

    set(rst_sources ${conf_docs}/index.rst
                    ${conf_docs}/muscleOptimizer.rst
                    ${conf_docs}/setupfile.rst
                    ${conf_docs}/guiplugin.rst
                    ${conf_docs}/installation.rst
                    ${conf_docs}/matlab.rst
                    ${conf_docs}/standalone.rst
    )
    set(doc_format_outputs "")
    set(doc_format_last "")
    foreach(format ${doc_formats})
      set(doc_format_output "doc_format_${format}")
      set(doc_format_log "build-${format}.log")
      add_custom_command(
        OUTPUT ${doc_format_output}
        COMMAND ${SPHINX_EXECUTABLE}
                -c ${CMAKE_CURRENT_BINARY_DIR}
                -d ${CMAKE_CURRENT_BINARY_DIR}/doctrees
                -b ${format}
                ${conf_docs}
                ${CMAKE_CURRENT_BINARY_DIR}/${format}
                > ${doc_format_log} # log stdout, pass stderr
        ${${format}_extra_commands}
        DEPENDS ${doc_format_last} ${rst_sources}
        COMMENT "sphinx-build ${format}: see ${CMAKE_BINARY_DIR}/docs/${doc_format_log}"
        VERBATIM
        )
      set_property(SOURCE ${doc_format_output} PROPERTY SYMBOLIC 1)
      list(APPEND doc_format_outputs ${doc_format_output})
      set(doc_format_last ${doc_format_output})

      if(${format} STREQUAL "latex")
        install(FILES   "${CMAKE_CURRENT_BINARY_DIR}/${format}/MuscleOptimizerHelp.pdf"
                DESTINATION "${CMAKE_INSTALL_DOCDIR}")
      else()
        install(DIRECTORY   "${CMAKE_CURRENT_BINARY_DIR}/${format}"
                DESTINATION "${CMAKE_INSTALL_DOCDIR}")
      endif()
    endforeach()

    add_custom_target(documentation ALL DEPENDS ${doc_format_outputs} SOURCES ${rst_sources})


    endif()
endif()

