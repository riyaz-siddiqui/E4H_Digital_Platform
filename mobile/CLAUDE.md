# E4H Field Assist Mobile (Flutter)

Flutter app for E4H field staff: installation reports (asset/BOM flow), AMC visits, assessments. Package name `selco`, Android ID `org.e4h.asset`.
Built on eGov DIGIT packages (`digit_ui_components`, `digit_forms_engine`, `digit_data_model`). Toolchain: Flutter 3.22.2 / Dart 3.4.3 (see `README.md`).

## Commands (run from `mobile/`)
- Codegen after touching `@freezed`, `@JsonSerializable`, `@MappableClass`, `@collection` (Isar) or `@RoutePage`:
  `dart run build_runner build --delete-conflicting-outputs`
- Check: `flutter analyze` then `flutter test` (or `flutter test test/<dir>/<file>_test.dart` for one file)

## Token rules — read before exploring
- NEVER read generated files: `*.g.dart`, `*.freezed.dart`, `*.gr.dart`, `*.mapper.dart`, `build/`, `.dart_tool/`. Read the source file and regenerate.
- Don't scan the codebase. Use the map below, then `grep` for a symbol. Read only the lines you need (offset/limit) for files > 300 lines (e.g. `lib/utils/background_service.dart` ~2000, `lib/utils/utils.dart` ~1150, `lib/repositories/dynamic_form_repo.dart`, `lib/utils/i18_key_constants.dart`, big pages like `lib/pages/amc_select_facility.dart`).
- Copy an existing example (listed below) instead of designing from scratch.
- Keep tasks small. If a request touches more than ~5 files or several screens, propose a split first.

## Code map
| Need | Go to | Example to copy |
|---|---|---|
| Screen | `lib/pages/` (flat; AMC = `amc_*.dart`, assessment = `assessment_*.dart`) — `@RoutePage()` | `lib/pages/amc_inbox.dart` |
| Route | path const in `lib/router/routes.dart`, entry in `lib/router/app_router.dart` | — |
| State | `lib/blocs/<feature>/<feature>.dart` — flutter_bloc + freezed events/states | `lib/blocs/scheduled_visit/scheduled_visit.dart` |
| Bloc providers | `lib/pages/authenticated.dart` | — |
| Data / API | `lib/repositories/<feature>_repo.dart`; endpoint path as a `const` in the repo method, assessment paths in `lib/repositories/assessment_api_paths.dart` | `lib/repositories/scheduled_visit_repo.dart` |
| Offline fallback | remote → Isar with a `fromCache` flag; shared checks (`isAuthenticationFailure`, `matchesFacilityName`) in `lib/repositories/cache_fallback.dart` | `scheduled_visit_repo.dart` |
| HTTP | `lib/data/remote_client.dart` (`DioClient().dio`), `lib/data/api_interceptors.dart`, `lib/data/network_manager.dart` | — |
| Offline cache | Isar collections in `lib/data/nosql/`; register the schema in `openIsar()` in `lib/utils/constants.dart`; instance via `await Constants().isar` | `cache_scheduled_visit.dart` |
| Background sync / submission jobs | `lib/utils/background_service.dart` (huge — grep, don't read) | — |
| Models | `lib/model/<feature>/` (freezed + json_serializable) | `lib/model/scheduled_visit/scheduled_visit.dart` |
| Shared widgets | `lib/widgets/<group>/` (cards, button, progress_indicator, images, customized_digit_widget…) | — |
| Helpers | `lib/utils/`; `context.translate` / `context.theme` in `lib/utils/extensions.dart`; logging via `AppLogger.instance` (`lib/utils/app_logger.dart`, never log PII — see `log_sanitizer.dart`) | — |
| Env | `lib/utils/envConfig.dart` reads `.env` (never print/commit its values) | — |
| Tests | `test/{blocs,data,model,repositories,utils,widgets}/`; Isar tests call `initializeOfflineIsar()` from `test/support/offline_isar.dart`; fakes in `test/support/` | `test/blocs/scheduled_visit_search_test.dart` |

## Conventions
- All user-visible text: `context.translate(i18.<group>.<key>)`, keys in `lib/utils/i18_key_constants.dart` (imported `as i18`). No hard-coded strings.
- Livelihood ports AMC screens from this app, so don't rename existing i18n key strings; mention behaviour changes to AMC in your summary.
- Backend not ready or unclear → stop and list it as a question for the backend team. Do not write backend code (`../backend/` is out of scope).
- Inject repositories through optional constructor params (`{ScheduledVisitRepository? repository}`, `{Dio? dio}`) so tests can pass fakes.
- Guard async UI/bloc work against stale responses (`_requestGeneration` counter, see `ScheduledVisitBloc`).
- Lists: loading, empty, error states + pull-to-refresh via `RefreshableFacilityList` / `FacilityListControls` (`lib/widgets/facility_list_controls.dart`). Safe area is applied app-wide (`AppSafeArea` in `main.dart`) — don't add another.
- Comments explain *why*. No ticket numbers in code or test names.
- Reuse existing helpers (`utils.dart`, `cache_fallback.dart`, `facility_list_filter.dart`, interceptor error codes) instead of adding copies.

## Done means
1. codegen re-run if annotations changed, 2. `flutter analyze` clean for touched files, 3. related tests pass / new test added for logic, 4. short summary: files changed + new i18n keys + anything left as a backend question.
