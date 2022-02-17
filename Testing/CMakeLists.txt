# Add standard remote object stores to user's
# configuration.
list(APPEND ExternalData_URL_TEMPLATES
  # Data published on GitHub Pages
  "https://insightsoftwareconsortium.github.io/ITKTestingData/%(algo)/%(hash)"

  # Data published on Girder
  "https://data.kitware.com:443/api/v1/file/hashsum/%(algo)/%(hash)/download"

  # Data published by developers using git-gerrit-push.
  "https://itk.org/files/ExternalData/%(algo)/%(hash)"
)

# Test checking that the program cannot run without the flag -f
add_test(NAME PolyDataCompressionFlag COMMAND $<TARGET_FILE:PolyDataCompression>
        -e ascii)
set_tests_properties(PolyDataCompressionFlag PROPERTIES WILL_FAIL true )

if(0) # TODO: these datasets are no longer available from the current download locations

# Test checking that the file compressed is properly written
ExternalData_Add_Test(PolyDataCompressionData NAME PolyDataCompressionCorrectOutput COMMAND $<TARGET_FILE:PolyDataCompression>
    -e binary
    -f DATA{Data/baseline.vtk}
  )
# Test of a normal run of the program with a VTK extension
add_executable(PolyDataCompressionTest vtkTest.cxx)
target_link_libraries(PolyDataCompressionTest PolyDataCompressionLib)

ExternalData_Add_Test(PolyDataCompressionData NAME PolyDataCompressionVTKExtension COMMAND $<TARGET_FILE:PolyDataCompressionTest>
  --compare
  DATA{Data/BaselineCingulumNormalRun-compressed.vtk}
  ${CMAKE_CURRENT_BINARY_DIR}/Data/CingulumNormalRun-compressed.vtk
  ModuleEntryPoint
  -e binary
  -x vtk
  -c 3
  -f DATA{Data/CingulumNormalRun.vtk}
)

# Test of a normal run of the program with a VTP extension
ExternalData_Add_Test(PolyDataCompressionData NAME PolyDataCompressionVTPExtension COMMAND $<TARGET_FILE:PolyDataCompressionTest>
  --compare
  DATA{Data/BaselineCingulumNormalRun-compressed.vtp}
  ${CMAKE_CURRENT_BINARY_DIR}/Data/CingulumNormalRun-compressed.vtp
  ModuleEntryPoint
  -e binary
  -x vtp
  -c 3
  -f DATA{Data/CingulumNormalRun.vtk}
)

#Test checking that a file in the current directory can be compressed
ExternalData_Expand_Arguments(PolyDataCompressionData expandedBaselinePath DATA{Data/baseline.vtk} )
get_filename_component( FileDirectory ${expandedBaselinePath} DIRECTORY)
ExternalData_Add_Test(PolyDataCompressionData NAME PolyDataCompressionCurrentDir WORKING_DIRECTORY ${FileDirectory} COMMAND $<TARGET_FILE:PolyDataCompression>
    -e binary
    -f baseline.vtk#We do not specify the path of the file with DATA{} since we want to test that PolyDataCompression works when giving a file with no path
  )
endif()

# Test checking that * character is expanded (only checking on UNIX systems). We search for bash
if( UNIX )
  find_program( bashPath bash )
  if( bashPath )
    ExternalData_Add_Test(PolyDataCompressionData NAME PolyDataCompressionStarExpansion COMMAND ${bashPath} -c
    "$<TARGET_FILE:PolyDataCompression> -e binary -f ${FileDirectory}/*.vtk"
    )
  endif()
endif()
# Test checking that the help can be printed even if no input file is given.
add_test(NAME PolyDataCompressionHelp COMMAND $<TARGET_FILE:PolyDataCompression>
    --help
  )

# Test checking that the version can be printed even if no input file is given.
add_test(NAME PolyDataCompressionVersion COMMAND $<TARGET_FILE:PolyDataCompression>
    --version
  )

set_tests_properties(PolyDataCompressionVersion PROPERTIES FAIL_REGULAR_EXPRESSION "Unspecified" )

# Add a build target to populate the real data.
ExternalData_Add_Target(PolyDataCompressionData)
