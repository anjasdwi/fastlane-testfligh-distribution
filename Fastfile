# fastlane/Fastfile

default_platform(:ios)

platform :ios do

  before_all do
    # Ensure we're on a clean state unless running in CI
    ensure_git_status_clean unless ENV['CI']

    # Setup CocoaPods
    cocoapods(
      clean_install: false,
      podfile: "./Podfile"
    )
  end

  desc "Build the app"
  lane :build do
    # Get API key for build number
    api_key = app_store_connect_api_key(
      key_id: ENV['APP_STORE_CONNECT_API_KEY_ID'],
      issuer_id: ENV['APP_STORE_CONNECT_ISSUER_ID'],
      key_content: ENV['APP_STORE_CONNECT_API_KEY_CONTENT'],
      is_key_content_base64: true,
      duration: 1200,
      in_house: false
    )

    # Get latest TestFlight build number and increment
    latest_build = latest_testflight_build_number(
      api_key: api_key,
      app_identifier: ENV['APP_IDENTIFIER'],
      initial_build_number: 1
    )
    new_build_number = latest_build + 1

    # Define xcconfig path
    xcconfig_path = "../Configuration/AppNameConfig-Dev.xcconfig"

    # Search BUILD_NUMBER in xcconfig and update it
    sh("sed -i '' 's/BUILD_NUMBER = .*/BUILD_NUMBER = #{new_build_number}/' #{xcconfig_path}")

    # Lookup version number
    version = get_version_number(xcodeproj: "AppName.xcodeproj", target: "AppName")
    build = new_build_number

    # Build app process
    build_app(
      workspace: "AppName.xcworkspace",  # IMPORTANT: Use workspace for CocoaPods
      scheme: "AppName-Dev",
      export_method: "app-store",
      output_directory: "./Builds",
      output_name: "AppName-Dev-(#{version})#{build}.ipa",
      clean: true,
      export_options: {
        provisioningProfiles: { 
          ENV['APP_IDENTIFIER_MAIN'] => "ci-cd_com.AppName.Dev",
          ENV['APP_IDENTIFIER_CONTENT'] => "ci-cd_com.AppName.Dev.NotificationContent",
          ENV['APP_IDENTIFIER_SERVICE'] => "ci-cd_com.AppName.Dev.NotificationService"
        }
      }
    )

    # Return new build number
    new_build_number
  end

  desc "Upload to TestFlight"
  lane :beta do
    # Define variables
    actual_build_number = build # Catch new_build_number from lane :build
    version = get_version_number(xcodeproj: "AppName.xcodeproj", target: "AppName")
    branch = ENV['CI_COMMIT_BRANCH'] || git_branch
    commit_hash = ENV['CI_COMMIT_SHORT_SHA'] || last_git_commit[:abbreviated_commit_hash]
    commit_message = ENV['CI_COMMIT_MESSAGE'] || last_git_commit[:message]
    triggered_by = ENV['GITLAB_USER_NAME'] || last_git_commit[:author]
    start_time = Time.now
    duration = Time.now - start_time
    pipeline_url = ENV['CI_PIPELINE_URL']

    # Get API key
    api_key = app_store_connect_api_key(
      key_id: ENV['APP_STORE_CONNECT_API_KEY_ID'],
      issuer_id: ENV['APP_STORE_CONNECT_ISSUER_ID'],
      key_content: ENV['APP_STORE_CONNECT_API_KEY_CONTENT'],
      is_key_content_base64: true,
      duration: 1200
    )

    # Generate changelog from git commits
    changelog = changelog_from_git_commits(
      commits_count: 10,
      pretty: "- %s",
      merge_commit_filtering: "exclude_merges"
    )

    # Construct IPA path
    ipa_file = "AppName-Dev-(#{version})#{actual_build_number}.ipa"
    ipa_path = "./Builds/#{ipa_file}"

    # Upload app process
    upload_to_testflight(
      api_key: api_key,
      skip_waiting_for_build_processing: true,
      skip_submission: false,
      distribute_external: false,
      notify_external_testers: false,
      changelog: changelog,
      ipa: ipa_path
    )

    # Send notification to Discord (if configured)
    if ENV['DISCORD_TESTFLIGHT_WEBHOOK_URL']
      discord_notifier(
        webhook_url: ENV['DISCORD_TESTFLIGHT_WEBHOOK_URL'],
        title: "🚀 iOS Build Success #{version} (#{actual_build_number})",
        thumbnail_url: "https://avatars.githubusercontent.com/u/11098337?s=280&v=4",
        description: <<~DESC
          App: AppName iOS - DEV
          Version: #{version} (#{actual_build_number})
          Branch: #{branch}
          Commit: #{commit_hash} - #{commit_message}
          Author: #{triggered_by}
          Duration: #{duration.round(2)} seconds
          Pipeline: #{pipeline_url}

          ✅ Build Completed
          👉 Install via TestFlight:
          https://testflight.apple.com/join/VnNqxr43
        DESC
      )
    end
  end

  # Error handling
  error do |lane, exception|
    if ENV['DISCORD_TESTFLIGHT_WEBHOOK_URL']
      discord_notifier(
        webhook_url: ENV['DISCORD_TESTFLIGHT_WEBHOOK_URL'],
        title: "Build Status",
        description: "❌ Error in lane '#{lane}': #{exception.message}",
        thumbnail_url: "https://avatars.githubusercontent.com/u/11098337?s=280&v=4"
      )
    end
  end
end