cmake_minimum_required(VERSION 3.15)

project(tcc_cmake)

add_executable(${PROJECT_NAME} main.c)

include(ExternalProject)

ExternalProject_Add(
	TinyCC

	GIT_REPOSITORY "https://github.com/TinyCC/tinycc"
	GIT_TAG "e9f59c804d41da9ba56493d5882fb75340d23c8e"

	CONFIGURE_COMMAND bash <SOURCE_DIR>/configure --prefix=<INSTALL_DIR>
	BUILD_COMMAND make

	UPDATE_COMMAND ""
	PATCH_COMMAND git checkout <SOURCE_DIR>/configure && git apply ${CMAKE_CURRENT_SOURCE_DIR}/0001-fix-out-of-tree-build-in-windows.patch
	TEST_COMMAND ""
)

ExternalProject_Get_Property(TinyCC INSTALL_DIR)

if (UNIX)
	target_include_directories(tcc_cmake PUBLIC ${INSTALL_DIR}/include)
	target_link_directories(tcc_cmake PUBLIC ${INSTALL_DIR}/lib)
elsif (WIN32)
	target_include_directories(tcc_cmake PUBLIC ${INSTALL_DIR}/libtcc)
	target_link_directories(tcc_cmake PUBLIC ${INSTALL_DIR})
endif()

target_link_libraries(tcc_cmake tcc pthread dl)


add_dependencies(tcc_cmake TinyCC)
install(TARGETS tcc_cmake DESTINATION bin)
install(DIRECTORY ${INSTALL_DIR}/include/ DESTINATION include)
install(DIRECTORY ${INSTALL_DIR}/lib/ DESTINATION lib)