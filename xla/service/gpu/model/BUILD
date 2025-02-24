# Libraries for performance modeling of HLO.
load("//xla/tests:build_defs.bzl", "xla_test")
load("@local_config_cuda//cuda:build_defs.bzl", "if_cuda")
load("//xla:xla.bzl", "xla_cc_test", "xla_nvml_deps")
load("@tsl//tsl:tsl.bzl", "internal_visibility")
load("@tsl//tsl:tsl.default.bzl", "get_compatible_with_portable")
load("@tsl//tsl/platform:build_config.bzl", "tf_proto_library")
load("@tsl//tsl/platform:build_config_root.bzl", "tf_cuda_tests_tags")
load("@tsl//tsl/platform/default:cuda_build_defs.bzl", "if_cuda_is_configured")

package(
    # copybara:uncomment default_applicable_licenses = ["//tensorflow:license"],
    default_visibility = internal_visibility([":friends"]),
    licenses = ["notice"],
)

package_group(
    name = "friends",
    includes = [
        "//xla:friends",
    ],
)

cc_library(
    name = "analytical_latency_estimator",
    srcs = ["analytical_latency_estimator.cc"],
    hdrs = ["analytical_latency_estimator.h"],
    deps = [
        ":gpu_collective_performance_model",
        ":gpu_hlo_cost_analysis",
        ":gpu_performance_model",
        ":gpu_performance_model_base",
        "//xla:xla_proto_cc",
        "//xla/hlo/ir:hlo",
        "//xla/hlo/utils:hlo_query",
        "//xla/service:hlo_cost_analysis",
        "//xla/service:latency_hiding_scheduler",
        "//xla/stream_executor:device_description",
        "@com_google_absl//absl/container:flat_hash_map",
        "@com_google_absl//absl/log",
        "@com_google_absl//absl/time",
        "@tsl//tsl/platform:status",
    ],
)

xla_test(
    name = "analytical_latency_estimator_test",
    srcs = ["analytical_latency_estimator_test.cc"],
    backend_tags = {"gpu": [
        "requires-gpu-sm70",
    ]},
    backends = [
        "gpu",
    ],
    deps = [
        ":analytical_latency_estimator",
        "//xla:shape_util",
        "//xla:statusor",
        "//xla/hlo/ir:hlo",
        "//xla/service:hlo_cost_analysis",
        "//xla/service:latency_hiding_scheduler",
        "//xla/service/gpu/tests:gpu_codegen_test",
        "//xla/stream_executor:device_description",
        "//xla/tests:xla_internal_test_main",
        "@com_google_absl//absl/status:statusor",
        "@com_google_absl//absl/strings",
        "@com_google_absl//absl/types:span",
        "@tsl//tsl/platform:statusor",
        "@tsl//tsl/platform:test",
    ],
)

cc_library(
    name = "fusion_analysis_cache",
    srcs = ["fusion_analysis_cache.cc"],
    hdrs = ["fusion_analysis_cache.h"],
    deps = [
        "//xla/hlo/ir:hlo",
        "//xla/service/gpu:hlo_fusion_analysis",
        "//xla/stream_executor:device_description",
        "@com_google_absl//absl/container:flat_hash_map",
        "@com_google_absl//absl/container:node_hash_map",
        "@com_google_absl//absl/synchronization",
    ],
)

xla_cc_test(
    name = "fusion_analysis_cache_test",
    srcs = ["fusion_analysis_cache_test.cc"],
    deps = [
        ":fusion_analysis_cache",
        "//xla/service:hlo_parser",
        "//xla/service/gpu:gpu_device_info_for_tests",
        "//xla/service/gpu:hlo_fusion_analysis",
        "//xla/stream_executor:device_description",
        "//xla/tests:hlo_test_base",
        "//xla/tests:xla_internal_test_main",
        "@com_google_absl//absl/strings:string_view",
        "@com_google_googletest//:gtest",
        "@tsl//tsl/platform:statusor",
    ],
)

cc_library(
    name = "gpu_cost_model_stats_collection",
    srcs = ["gpu_cost_model_stats_collection.cc"],
    hdrs = ["gpu_cost_model_stats_collection.h"],
    deps = [
        ":gpu_hlo_cost_analysis",
        ":gpu_performance_model",
        ":gpu_performance_model_base",
        "//xla:statusor",
        "//xla/hlo/ir:hlo",
        "//xla/service:hlo_cost_analysis",
        "//xla/service:hlo_graph_dumper",
        "//xla/service:hlo_pass",
        "//xla/stream_executor:device_description",
        "@com_google_absl//absl/container:flat_hash_set",
        "@com_google_absl//absl/status:statusor",
        "@com_google_absl//absl/strings",
        "@tsl//tsl/platform:status",
    ],
)

xla_cc_test(
    name = "gpu_cost_model_stats_collection_test",
    srcs = ["gpu_cost_model_stats_collection_test.cc"],
    deps = [
        ":gpu_cost_model_stats_collection",
        ":gpu_hlo_cost_analysis",
        "//xla:shape_util",
        "//xla/hlo/ir:hlo",
        "//xla/service:hlo_cost_analysis",
        "//xla/service/gpu:backend_configs_cc",
        "//xla/service/gpu:gpu_device_info_for_tests",
        "//xla/tests:hlo_test_base",
        "//xla/tests:verified_hlo_module",
        "//xla/tests:xla_internal_test_main",
        "@com_google_googletest//:gtest",
        "@tsl//tsl/platform:statusor",
    ],
)

cc_library(
    name = "gpu_hlo_cost_analysis",
    srcs = ["gpu_hlo_cost_analysis.cc"],
    hdrs = ["gpu_hlo_cost_analysis.h"],
    compatible_with = get_compatible_with_portable(),
    deps = [
        ":hlo_op_profile_proto_cc",
        ":hlo_op_profiles",
        "//xla:shape_util",
        "//xla:status",
        "//xla:util",
        "//xla/hlo/ir:hlo",
        "//xla/service:collective_ops_utils",
        "//xla/service:elemental_ir_emitter",
        "//xla/service:hlo_cost_analysis",
        "//xla/service:hlo_module_config",
        "//xla/service/gpu:backend_configs_cc",
        "//xla/service/gpu:cublas_cudnn",
        "//xla/stream_executor:device_description",
        "@com_google_absl//absl/algorithm:container",
        "@com_google_absl//absl/container:flat_hash_map",
        "@com_google_absl//absl/container:flat_hash_set",
        "@com_google_absl//absl/log",
        "@com_google_absl//absl/log:check",
        "@com_google_absl//absl/status",
        "@com_google_absl//absl/strings",
        "@tsl//tsl/platform:errors",
        "@tsl//tsl/platform:statusor",
    ],
)

xla_cc_test(
    name = "gpu_hlo_cost_analysis_test",
    srcs = ["gpu_hlo_cost_analysis_test.cc"],
    deps = [
        ":gpu_hlo_cost_analysis",
        "//xla:shape_util",
        "//xla:test_helpers",
        "//xla/hlo/ir:hlo",
        "//xla/service:hlo_cost_analysis",
        "//xla/tests:hlo_test_base",
        "//xla/tests:xla_internal_test_main",
        "@com_google_absl//absl/strings:string_view",
        "@com_google_googletest//:gtest",
        "@tsl//tsl/platform:statusor",
    ],
)

cc_library(
    name = "gpu_performance_model_base",
    srcs = ["gpu_performance_model_base.cc"],
    hdrs = ["gpu_performance_model_base.h"],
    deps = [
        ":fusion_analysis_cache",
        ":gpu_hlo_cost_analysis",
        "//xla:shape_util",
        "//xla:util",
        "//xla/hlo/ir:hlo",
        "//xla/service/gpu:backend_configs_cc",
        "//xla/service/gpu:hlo_fusion_analysis",
        "//xla/service/gpu:hlo_traversal",
        "//xla/service/gpu:launch_dimensions",
        "//xla/service/gpu/fusions",
        "//xla/service/gpu/fusions:fusion_emitter",
        "//xla/stream_executor:device_description",
        "@com_google_absl//absl/container:flat_hash_map",
        "@com_google_absl//absl/container:flat_hash_set",
        "@com_google_absl//absl/log",
        "@com_google_absl//absl/log:check",
        "@com_google_absl//absl/synchronization",
        "@com_google_absl//absl/time",
    ],
)

xla_cc_test(
    name = "gpu_performance_model_base_test",
    srcs = ["gpu_performance_model_base_test.cc"],
    deps = [
        ":gpu_hlo_cost_analysis",
        ":gpu_performance_model_base",
        "//xla:shape_util",
        "//xla:test_helpers",
        "//xla/hlo/ir:hlo",
        "//xla/service/gpu:backend_configs_cc",
        "//xla/service/gpu:gpu_device_info_for_tests",
        "//xla/stream_executor:device_description",
        "//xla/tests:hlo_test_base",
        "//xla/tests:xla_internal_test_main",
        "@com_google_absl//absl/strings",
        "@com_google_googletest//:gtest",
        "@tsl//tsl/platform:statusor",
    ],
)

cc_library(
    name = "gpu_performance_model",
    srcs = ["gpu_performance_model.cc"],
    hdrs = ["gpu_performance_model.h"],
    deps = [
        ":coalescing_analysis",
        ":gpu_hlo_cost_analysis",
        ":gpu_performance_model_base",
        "//xla:shape_util",
        "//xla:util",
        "//xla/hlo/ir:hlo",
        "//xla/service/gpu:backend_configs_cc",
        "//xla/service/gpu:hlo_fusion_analysis",
        "//xla/service/gpu:launch_dimensions",
        "//xla/stream_executor:device_description",
        "@com_google_absl//absl/log",
        "@com_google_absl//absl/log:check",
        "@com_google_absl//absl/time",
        "@com_google_absl//absl/types:span",
        "@llvm-project//llvm:Support",
        "@tsl//tsl/platform:status",
    ],
)

xla_cc_test(
    name = "gpu_performance_model_test",
    srcs = ["gpu_performance_model_test.cc"],
    deps = [
        ":gpu_hlo_cost_analysis",
        ":gpu_indexing_performance_model",
        ":gpu_performance_model",
        ":gpu_performance_model_base",
        "//xla:shape_util",
        "//xla:test_helpers",
        "//xla/hlo/ir:hlo",
        "//xla/service:hlo_module_config",
        "//xla/service/gpu:backend_configs_cc",
        "//xla/service/gpu:gpu_device_info_for_tests",
        "//xla/stream_executor:device_description",
        "//xla/tests:hlo_test_base",
        "//xla/tests:xla_internal_test_main",
        "@com_google_absl//absl/status:statusor",
        "@com_google_absl//absl/strings",
        "@com_google_absl//absl/time",
        "@com_google_googletest//:gtest",
        "@llvm-project//mlir:IR",
        "@tsl//tsl/platform:errors",
        "@tsl//tsl/platform:statusor",
    ],
)

cc_library(
    name = "gpu_collective_performance_model",
    srcs = ["gpu_collective_performance_model.cc"],
    hdrs = ["gpu_collective_performance_model.h"],
    local_defines = if_cuda_is_configured(["GOOGLE_CUDA=1"]),
    deps = [
        ":coalescing_analysis",
        ":fusion_analysis_cache",
        ":gpu_hlo_cost_analysis",
        ":gpu_performance_model_base",
        ":hlo_op_profiles",
        ":indexing_analysis",
        ":indexing_map",
        "//xla:shape_util",
        "//xla:statusor",
        "//xla:util",
        "//xla/hlo/ir:hlo",
        "//xla/service:hlo_cost_analysis",
        "//xla/service:hlo_dataflow_analysis",
        "//xla/service/gpu:backend_configs_cc",
        "//xla/service/gpu:gpu_fusible",
        "//xla/service/gpu:hlo_fusion_analysis",
        "//xla/service/gpu:hlo_traversal",
        "//xla/service/gpu:launch_dimensions",
        "//xla/service/gpu/fusions",
        "//xla/service/gpu/fusions:fusion_emitter",
        "//xla/stream_executor:device_description",
        "@com_google_absl//absl/container:flat_hash_map",
        "@com_google_absl//absl/container:flat_hash_set",
        "@com_google_absl//absl/log",
        "@com_google_absl//absl/log:check",
        "@com_google_absl//absl/strings",
        "@com_google_absl//absl/synchronization",
        "@com_google_absl//absl/time",
        "@com_google_absl//absl/types:span",
        "@llvm-project//llvm:Support",
        "@llvm-project//mlir:IR",
        "@tsl//tsl/platform:status",
    ] + if_cuda_is_configured(xla_nvml_deps()),
)

xla_cc_test(
    name = "gpu_collective_performance_model_test",
    srcs = ["gpu_collective_performance_model_test.cc"],
    deps = [
        ":gpu_collective_performance_model",
        "//xla/service/gpu:backend_configs_cc",
        "//xla/tests:hlo_test_base",
        "//xla/tests:xla_internal_test_main",
        "@com_google_googletest//:gtest",
    ],
)

cc_library(
    name = "gpu_indexing_performance_model",
    srcs = ["gpu_indexing_performance_model.cc"],
    hdrs = ["gpu_indexing_performance_model.h"],
    deps = [
        ":coalescing_analysis",
        ":gpu_hlo_cost_analysis",
        ":gpu_performance_model_base",
        ":hlo_op_profiles",
        ":indexing_analysis",
        ":indexing_map",
        "//xla:shape_util",
        "//xla:util",
        "//xla/hlo/ir:hlo",
        "//xla/service:hlo_cost_analysis",
        "//xla/service/gpu:backend_configs_cc",
        "//xla/service/gpu:hlo_fusion_analysis",
        "//xla/service/gpu:hlo_traversal",
        "//xla/service/gpu:launch_dimensions",
        "//xla/stream_executor:device_description",
        "@com_google_absl//absl/log",
        "@com_google_absl//absl/log:check",
        "@com_google_absl//absl/time",
        "@com_google_absl//absl/types:span",
        "@llvm-project//mlir:IR",
        "@tsl//tsl/platform:status",
    ],
)

xla_cc_test(
    name = "gpu_indexing_performance_model_test",
    srcs = ["gpu_indexing_performance_model_test.cc"],
    deps = [
        ":gpu_hlo_cost_analysis",
        ":gpu_indexing_performance_model",
        ":gpu_performance_model_base",
        "//xla:shape_util",
        "//xla/hlo/ir:hlo",
        "//xla/service/gpu:backend_configs_cc",
        "//xla/service/gpu:gpu_device_info_for_tests",
        "//xla/stream_executor:device_description",
        "//xla/tests:hlo_test_base",
        "//xla/tests:xla_internal_test_main",
        "@com_google_absl//absl/strings",
        "@com_google_absl//absl/time",
        "@com_google_googletest//:gtest",
        "@llvm-project//mlir:IR",
        "@tsl//tsl/platform:statusor",
    ],
)

cc_library(
    name = "affine_map_printer",
    srcs = ["affine_map_printer.cc"],
    hdrs = ["affine_map_printer.h"],
    deps = [
        "@com_google_absl//absl/strings",
        "@com_google_absl//absl/types:span",
        "@llvm-project//llvm:Support",
        "@llvm-project//mlir:IR",
        "@llvm-project//mlir:Support",
    ],
)

xla_cc_test(
    name = "affine_map_printer_test",
    srcs = ["affine_map_printer_test.cc"],
    deps = [
        ":affine_map_printer",
        "//xla/tests:hlo_test_base",
        "//xla/tests:xla_internal_test_main",
        "@com_google_googletest//:gtest",
        "@llvm-project//mlir:IR",
        "@tsl//tsl/platform:test",
    ],
)

cc_library(
    name = "indexing_map",
    srcs = ["indexing_map.cc"],
    hdrs = [
        "indexing_context.h",
        "indexing_map.h",
    ],
    deps = [
        ":affine_map_printer",
        "//xla/hlo/ir:hlo",
        "@com_google_absl//absl/container:flat_hash_map",
        "@com_google_absl//absl/types:span",
        "@llvm-project//llvm:Support",
        "@llvm-project//mlir:IR",
        "@llvm-project//mlir:Support",
        "@tsl//tsl/platform:logging",
    ],
)

xla_cc_test(
    name = "indexing_map_test",
    srcs = ["indexing_map_test.cc"],
    deps = [
        ":affine_map_printer",
        ":indexing_map",
        ":indexing_test_utils",
        "//xla/tests:hlo_test_base",
        "//xla/tests:xla_internal_test_main",
        "@com_google_googletest//:gtest",
        "@llvm-project//mlir:IR",
        "@tsl//tsl/platform:test",
    ],
)

cc_library(
    name = "indexing_test_utils",
    testonly = True,
    srcs = ["indexing_test_utils.cc"],
    hdrs = ["indexing_test_utils.h"],
    deps = [
        ":indexing_analysis",
        ":indexing_map",
        "//xla/hlo/ir:hlo",
        "//xla/tests:hlo_test_base",
        "//xla/tests:verified_hlo_module",
        "@com_google_absl//absl/container:flat_hash_set",
        "@com_google_absl//absl/log:check",
        "@com_google_absl//absl/strings",
        "@com_google_absl//absl/strings:string_view",
        "@com_google_googletest//:gtest",
        "@llvm-project//llvm:Support",
        "@llvm-project//mlir:AsmParser",
        "@llvm-project//mlir:IR",
    ],
)

cc_library(
    name = "indexing_analysis",
    srcs = ["indexing_analysis.cc"],
    hdrs = ["indexing_analysis.h"],
    deps = [
        ":affine_map_printer",
        ":indexing_map",
        "//xla:permutation_util",
        "//xla:shape_util",
        "//xla:util",
        "//xla:xla_data_proto_cc",
        "//xla/hlo/ir:hlo",
        "//xla/service:gather_simplifier",
        "//xla/service/gpu:hlo_traversal",
        "//xla/service/gpu:matmul_utils",
        "//xla/service/gpu/fusions:tiling_util",
        "@com_google_absl//absl/algorithm:container",
        "@com_google_absl//absl/container:flat_hash_map",
        "@com_google_absl//absl/container:flat_hash_set",
        "@com_google_absl//absl/container:inlined_vector",
        "@com_google_absl//absl/log:check",
        "@com_google_absl//absl/types:span",
        "@llvm-project//llvm:Support",
        "@llvm-project//mlir:IR",
        "@llvm-project//mlir:Support",
    ],
)

xla_cc_test(
    name = "indexing_analysis_test",
    srcs = ["indexing_analysis_test.cc"],
    deps = [
        ":indexing_analysis",
        ":indexing_map",
        ":indexing_test_utils",
        "//xla/hlo/ir:hlo",
        "//xla/service/gpu:hlo_traversal",
        "//xla/service/gpu/fusions:tiling_util",
        "//xla/tests:hlo_test_base",
        "//xla/tests:xla_internal_test_main",
        "@com_google_absl//absl/strings:string_view",
        "@com_google_googletest//:gtest",
        "@llvm-project//mlir:IR",
        "@tsl//tsl/platform:test",
    ],
)

cc_library(
    name = "tile_analysis",
    srcs = ["tile_analysis.cc"],
    hdrs = ["tile_analysis.h"],
    deps = [
        ":affine_map_printer",
        ":indexing_map",
        "@com_google_absl//absl/algorithm:container",
        "@com_google_absl//absl/log:check",
        "@com_google_absl//absl/strings",
        "@llvm-project//llvm:Support",
        "@llvm-project//mlir:IR",
        "@llvm-project//mlir:Support",
        "@tsl//tsl/platform:status",
    ],
)

xla_cc_test(
    name = "tile_analysis_test",
    srcs = ["tile_analysis_test.cc"],
    deps = [
        ":affine_map_printer",
        ":indexing_analysis",
        ":indexing_test_utils",
        ":tile_analysis",
        "//xla/hlo/ir:hlo",
        "//xla/tests:hlo_test_base",
        "//xla/tests:xla_internal_test_main",
        "@com_google_absl//absl/strings",
        "@com_google_absl//absl/strings:string_view",
        "@com_google_googletest//:gtest",
        "@llvm-project//mlir:IR",
        "@tsl//tsl/platform:test",
    ],
)

cc_library(
    name = "symbolic_tile_analysis",
    srcs = ["symbolic_tile_analysis.cc"],
    hdrs = ["symbolic_tile_analysis.h"],
    deps = [
        ":indexing_analysis",
        ":indexing_map",
        ":tile_analysis",
        "//xla:shape_util",
        "//xla:status",
        "//xla/hlo/ir:hlo",
        "//xla/service:instruction_fusion",
        "@com_google_absl//absl/container:flat_hash_map",
        "@com_google_absl//absl/container:flat_hash_set",
        "@com_google_absl//absl/log:check",
        "@com_google_absl//absl/strings",
        "@com_google_absl//absl/types:span",
        "@llvm-project//llvm:Support",
        "@llvm-project//mlir:IR",
    ],
)

xla_cc_test(
    name = "symbolic_tile_analysis_test",
    srcs = ["symbolic_tile_analysis_test.cc"],
    deps = [
        ":indexing_map",
        ":symbolic_tile_analysis",
        "//xla/hlo/ir:hlo",
        "//xla/tests:hlo_test_base",
        "//xla/tests:verified_hlo_module",
        "@com_google_absl//absl/strings",
        "@com_google_absl//absl/types:span",
        "@com_google_googletest//:gtest_main",
        "@llvm-project//mlir:IR",
        "@tsl//tsl/platform:statusor",
    ],
)

cc_library(
    name = "coalescing_analysis",
    srcs = ["coalescing_analysis.cc"],
    hdrs = ["coalescing_analysis.h"],
    deps = [
        ":indexing_analysis",
        ":indexing_map",
        "//xla:shape_util",
        "//xla/hlo/ir:hlo",
        "//xla/service/gpu:gpu_fusible",
        "//xla/service/gpu:hlo_fusion_analysis",
        "//xla/service/gpu:hlo_traversal",
        "//xla/service/gpu/fusions:fusion_emitter",
        "@com_google_absl//absl/container:flat_hash_map",
        "@com_google_absl//absl/container:inlined_vector",
        "@com_google_absl//absl/log:check",
        "@com_google_absl//absl/types:span",
        "@llvm-project//llvm:Support",
        "@llvm-project//mlir:IR",
        "@llvm-project//mlir:Support",
    ],
)

xla_cc_test(
    name = "coalescing_analysis_test",
    srcs = ["coalescing_analysis_test.cc"],
    deps = [
        ":coalescing_analysis",
        ":indexing_map",
        "//xla:shape_util",
        "//xla/hlo/ir:hlo",
        "//xla/service:hlo_module_config",
        "//xla/service/gpu:gpu_device_info_for_tests",
        "//xla/service/gpu:hlo_fusion_analysis",
        "//xla/service/gpu:hlo_traversal",
        "//xla/service/gpu/fusions",
        "//xla/service/gpu/fusions:fusion_emitter",
        "//xla/stream_executor:device_description",
        "//xla/tests:hlo_test_base",
        "@com_google_absl//absl/strings:string_view",
        "@com_google_googletest//:gtest_main",
        "@llvm-project//mlir:IR",
        "@tsl//tsl/platform:test",
    ],
)

tf_proto_library(
    name = "hlo_op_profile_proto",
    srcs = ["hlo_op_profile.proto"],
    cc_api_version = 2,
    make_default_target_header_only = True,
    protodeps = [
        "//xla/service:hlo_proto",
    ],
)

cc_library(
    name = "hlo_op_profiles",
    srcs = ["hlo_op_profiles.cc"],
    hdrs = [
        "hlo_op_profiles.h",
        "hlo_op_profiles_data.h",
    ],
    compatible_with = get_compatible_with_portable(),
    deps = [
        ":hlo_op_profile_proto_cc",
        "//xla:types",
        "//xla:xla_data_proto_cc",
        "//xla/hlo/ir:hlo",
        "//xla/service:hlo_proto_cc",
        "//xla/stream_executor:device_description",
        "@com_google_absl//absl/container:flat_hash_map",
        "@com_google_absl//absl/memory",
        "@com_google_absl//absl/strings",
        "@tsl//tsl/platform:logging",
        "@tsl//tsl/platform:protobuf",
    ],
)

xla_cc_test(
    name = "hlo_op_profiles_test",
    srcs = ["hlo_op_profiles_test.cc"],
    deps = [
        ":hlo_op_profiles",
        "//xla:xla_data_proto_cc",
        "//xla/hlo/ir:hlo",
        "//xla/service/gpu:gpu_device_info_for_tests",
        "//xla/tests:hlo_test_base",
        "@com_google_googletest//:gtest_main",
    ],
)

cc_library(
    name = "hlo_op_profiler_lib",
    testonly = True,
    srcs = ["hlo_op_profiler.cc"],
    hdrs = ["hlo_op_profiler.h"],
    local_defines = if_cuda(["GOOGLE_CUDA"]),
    deps = [
        ":hlo_op_profile_proto_cc",
        "//xla:debug_options_flags",
        "//xla:literal",
        "//xla:shape_util",
        "//xla:statusor",
        "//xla:util",
        "//xla:xla_data_proto_cc",
        "//xla/backends/profiler/gpu:cupti_collector",
        "//xla/backends/profiler/gpu:cupti_tracer",
        "//xla/hlo/ir:hlo",
        "//xla/service:executable",
        "//xla/service:gpu_plugin",
        "//xla/service:hlo_module_config",
        "//xla/service:hlo_runner",
        "//xla/service:interpreter_plugin",
        "//xla/stream_executor:device_description",
        "//xla/tests:test_utils",
        "@com_google_absl//absl/log",
        "@com_google_absl//absl/log:check",
        "@com_google_absl//absl/status:statusor",
        "@com_google_absl//absl/time",
        "@tsl//tsl/platform:errors",
        "@tsl//tsl/platform:statusor",
    ],
)

[
    xla_cc_test(
        name = "hlo_op_profiler_run_" + sm,
        timeout = "eternal",
        srcs = ["hlo_op_profiler_run.cc"],
        # Disable backend optimizations (in particular reassociate and instcombine) which would optimize
        # expressions like integer add and multiply.
        args = ["--xla_backend_optimization_level=0"],
        # This is a development tool, not a normal test, and thus should only be run
        # manually with --config=cuda.
        tags = [
            "gpu",
            "manual",
            "notap",
            "requires-gpu-" + sm + "-only",
        ],
        deps = [
            ":hlo_op_profile_proto_cc",
            ":hlo_op_profiler_lib",
            ":hlo_op_profiles",
            "//xla:debug_options_flags",
            "//xla:xla_data_proto_cc",
            "//xla/hlo/ir:hlo",
            "//xla/service:hlo_runner",
            "//xla/service:platform_util",
            "//xla/stream_executor:device_description",
            "@com_google_absl//absl/log",
            "@com_google_absl//absl/strings",
            "@com_google_absl//absl/strings:str_format",
            "@tsl//tsl/platform:env",
            "@tsl//tsl/platform:path",
            "@tsl//tsl/platform:platform_port",
            "@tsl//tsl/platform:status",
            "@tsl//tsl/util:command_line_flags",
        ],
    )
    for sm in [
        "sm60",
        "sm70",
        "sm80",
        "sm90",
    ]
]

xla_cc_test(
    name = "hlo_op_profiler_test",
    srcs = ["hlo_op_profiler_test.cc"],
    local_defines = if_cuda(["GOOGLE_CUDA"]),
    tags = tf_cuda_tests_tags(),
    deps = [
        ":hlo_op_profiler_lib",
        "//xla/hlo/ir:hlo",
        "//xla/service:gpu_plugin",
        "//xla/tests:hlo_test_base",
        "@com_google_googletest//:gtest",
        "@tsl//tsl/platform:test_main",
    ],
)
