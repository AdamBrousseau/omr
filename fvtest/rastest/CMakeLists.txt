###############################################################################
# Copyright (c) 2017, 2020 IBM Corp. and others
#
# This program and the accompanying materials are made available under
# the terms of the Eclipse Public License 2.0 which accompanies this
# distribution and is available at http://eclipse.org/legal/epl-2.0
# or the Apache License, Version 2.0 which accompanies this distribution
# and is available at https://www.apache.org/licenses/LICENSE-2.0.
#
# This Source Code may also be made available under the following Secondary
# Licenses when the conditions for such availability set forth in the
# Eclipse Public License, v. 2.0 are satisfied: GNU General Public License,
# version 2 with the GNU Classpath Exception [1] and GNU General Public
# License, version 2 with the OpenJDK Assembly Exception [2].
#
# [1] https://www.gnu.org/software/classpath/license.html
# [2] http://openjdk.java.net/legal/assembly-exception.html
#
# SPDX-License-Identifier: EPL-2.0 OR Apache-2.0 OR GPL-2.0 WITH Classpath-exception-2.0 OR LicenseRef-GPL-2.0 WITH Assembly-exception
###############################################################################

if(OMR_OS_AIX)
	# Ideally we could just set BUILD_RPATH but that is not available on cmake 3.4
	set(CMAKE_INSTALL_RPATH ".")
	set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
endif()

add_tracegen(omr_test.tdf)
include_directories(${CMAKE_CURRENT_BINARY_DIR})

add_library(bindthreadagent SHARED
	bindthreadagent.c
	omragent.def
)

target_link_libraries(bindthreadagent PUBLIC omrtestutil)

omr_add_exports(bindthreadagent
	OMRAgent_OnLoad
	OMRAgent_OnUnload
)

add_library(memorycategoriesagent SHARED
	memorycategoriesagent.c
	omragent.def
)

target_link_libraries(memorycategoriesagent PRIVATE omrtestutil)

omr_add_exports(memorycategoriesagent
	OMRAgent_OnLoad
	OMRAgent_OnUnload
)

add_library(traceagent SHARED
	traceagent.c
	omragent.def
)

target_link_libraries(traceagent PRIVATE omrtestutil)

omr_add_exports(traceagent
	OMRAgent_OnLoad
	OMRAgent_OnUnload
)

add_library(traceNotStartedAgent SHARED
	traceNotStartedAgent.c
	omragent.def
)

target_link_libraries(traceNotStartedAgent PRIVATE omrtestutil)

omr_add_exports(traceNotStartedAgent
	OMRAgent_OnLoad
	OMRAgent_OnUnload
)

add_library(cpuLoadAgent SHARED
	cpuLoadAgent.c
	omragent.def
)

target_link_libraries(cpuLoadAgent PUBLIC omrtestutil)

omr_add_exports(cpuLoadAgent
	OMRAgent_OnLoad
	OMRAgent_OnUnload
)

add_library(invalidAgentMissingOnLoad SHARED
	invalidAgentMissingOnLoad.c
	invalidAgentMissingOnLoad.def
)

target_link_libraries(invalidAgentMissingOnLoad PUBLIC omr_base)

omr_add_exports(invalidAgentMissingOnLoad
	OMRAgent_OnUnload
)

add_library(invalidAgentMissingOnUnload SHARED
	invalidAgentMissingOnUnload.c
	invalidAgentMissingOnUnload.def
)

target_link_libraries(invalidAgentMissingOnUnload PUBLIC omr_base)

omr_add_exports(invalidAgentMissingOnUnload
	OMRAgent_OnLoad
)

add_library(invalidAgentReturnError SHARED
	invalidAgentReturnError.c
	omragent.def
)

target_link_libraries(invalidAgentReturnError PUBLIC omr_base)

omr_add_exports(invalidAgentReturnError
	OMRAgent_OnLoad
	OMRAgent_OnUnload
)

add_library(sampleSubscriber SHARED
	sampleSubscriber.c
	omragent.def
)

target_link_libraries(sampleSubscriber PUBLIC omr_base)

omr_add_exports(sampleSubscriber
	OMRAgent_OnLoad
	OMRAgent_OnUnload
)

add_library(subscriberAgent SHARED
	subscriberAgent.c
	omragent.def
)
target_link_libraries(subscriberAgent PRIVATE omrtestutil)

omr_add_exports(subscriberAgent
	OMRAgent_OnLoad
	OMRAgent_OnUnload
)

add_library(subscriberAgentWithJ9thread SHARED
	subscriberAgentWithJ9thread.c
	omragent.def
)
target_link_libraries(subscriberAgentWithJ9thread PRIVATE omrtestutil)

omr_add_exports(subscriberAgentWithJ9thread
	OMRAgent_OnLoad
	OMRAgent_OnUnload
)

add_library(traceOptionAgent SHARED
	traceOptionAgent.c
	omragent.def
)

target_link_libraries(traceOptionAgent
	PUBLIC
		omr_base
)

omr_add_exports(traceOptionAgent
	OMRAgent_OnLoad
	OMRAgent_OnUnload
)

add_executable(omrrastest
	agentNegativeTest.cpp
	agentTest.cpp
	main.cpp
	memoryCategoriesTest.cpp
	methodDictionaryTest.cpp
	rasTestHelpers.cpp
	traceLifecycleTest.cpp
	traceLogTest.cpp
	traceRecordHelpers.cpp
	traceTest.cpp
	ut_omr_test.c
)
target_link_libraries(omrrastest
	PRIVATE
		omrGtestGlue
		omrtestutil
		omrcore
		omrvmstartup
		${OMR_PORT_LIB}
)

add_dependencies(omrrastest
	bindthreadagent
	cpuLoadAgent
	invalidAgentMissingOnLoad
	invalidAgentReturnError
	memorycategoriesagent
	sampleSubscriber
	traceagent
	traceNotStartedAgent
)

add_executable(omrsubscribertest
	main.cpp
	rasTestHelpers.cpp
	subscriberTest.cpp
	ut_omr_test.c
)
if(OMR_THR_FORK_SUPPORT)
	target_sources(omrsubscribertest PRIVATE subscriberForkTest.cpp)
endif()
target_link_libraries(omrsubscribertest
	PRIVATE
		omrGtestGlue
		omrtestutil
		omrcore
		omrvmstartup
		${OMR_PORT_LIB}
)
add_dependencies(omrsubscribertest
	subscriberAgent
	subscriberAgentWithJ9thread
)

add_executable(omrtraceoptiontest
	main.cpp
	rasTestHelpers.cpp
	traceOptionTest.cpp
	ut_omr_test.c
)
target_link_libraries(omrtraceoptiontest
	PRIVATE
		omrGtestGlue
		omrtestutil
		omrcore
		omrvmstartup
		${OMR_PORT_LIB}
)
add_dependencies(omrtraceoptiontest
	traceOptionAgent
)

add_test(NAME rastest COMMAND omrrastest --gtest_output=xml:${CMAKE_CURRENT_BINARY_DIR}/omrrastest-results.xml)
add_test(NAME subscribertest COMMAND omrsubscribertest --gtest_output=xml:${CMAKE_CURRENT_BINARY_DIR}/omrsubscribertest-results.xml)
add_test(NAME traceoptiontest COMMAND omrtraceoptiontest --gtest_output=xml:${CMAKE_CURRENT_BINARY_DIR}/omrtraceoptiontest-results.xml)

set_target_properties(
	omrrastest
	omrsubscribertest
	omrtraceoptiontest
	PROPERTIES
	FOLDER "fvtest/rastest"
)

set_target_properties(
	bindthreadagent
	cpuLoadAgent
	invalidAgentMissingOnLoad
	invalidAgentMissingOnUnload
	invalidAgentReturnError
	memorycategoriesagent
	sampleSubscriber
	subscriberAgent
	subscriberAgentWithJ9thread
	traceagent
	traceNotStartedAgent
	traceOptionAgent

	PROPERTIES
	FOLDER "fvtest/rastest/agents"
)
