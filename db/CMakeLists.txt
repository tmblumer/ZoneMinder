# CMakeLists.txt for the ZoneMinder database scripts

# Create files from the .in files
configure_file(zm_create.sql.in ${CMAKE_CURRENT_SOURCE_DIR}/zm_create.sql @ONLY)

# Install the database scripts, exclude makefiles and cmake stuff
install(DIRECTORY . DESTINATION ${CMAKE_INSTALL_DATADIR}/zoneminder/db FILES_MATCHING PATTERN "zm*.sql" PATTERN "*Make*" EXCLUDE)

