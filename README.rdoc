See http://github.com/alloy/UISpec/blob/master/installation.txt for the UISpec install file.


An example of a Kicker (http://github.com/alloy/kicker) script to build and run specs continuously:

  require 'tmpdir'

  SDK_DIR = "/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator4.0.sdk"
  TMP_USER_HOME = Dir.tmpdir

  def with_env(env)
    before = env.inject({}) { |h, (k, _)| h[k] = ENV[k]; h }
    env.each { |k, v| ENV[k] = v }
    yield
  ensure
    before.each { |k, v| ENV[k] = v }
  end

  def run_specs
    if system("xcodebuild -project Project.xcodeproj -target UISpecs -configuration Debug -sdk /Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator4.0.sdk > /dev/null")
      with_env('DYLD_ROOT_PATH' => SDK_DIR, 'IPHONE_SIMULATOR_ROOT' => SDK_DIR, 'CFFIXED_USER_HOME' => TMP_USER_HOME) do
        system "./build/Debug-iphonesimulator/UISpecs.app/UISpecs -RegisterForSystemEvents"
      end
    else
      puts "[!] Failed to build UISpecs.app"
    end
  end

  process do |files|
    run_specs if files.any? { |file| file =~ /Classes|Spec/ }
    files.clear
  end