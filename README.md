# sency-motion-android-demo

### [Adding dependencies](#adding-dependencies)
### [Initialization](#initialization)
### [Usage](#usage)
* [Summary result data response](#summary-result-data-response)
* [Start workout](#start-workout)
* [Start custom workout](#start-custom-workout)
* [Custom phone moved dialog](#custom-phone-moved-dialog)
* [Get workout info](#get-workout-info)

### Adding dependencies
Here is the current available version of the SencyMotion project:

| Project        | Version |
|----------------|:-------:|
| sencymotionlib | 0.0.23  |


To use SencyMotion in your project, add these dependencies to your application in your `build.gradle` file.
```groovy
dependencies {
    implementation 'com.sency.sencymotion:sencymotionlib:$latest_version'
}
```

### Initialization
Initialize SencyMotion in your application's `Application` class.
```kotlin
class App : Application() {

    lateinit var sencyMotion: SencyMotion

    override fun onCreate() {
        super.onCreate()
        sencyMotion = SencyMotion.Configuration(applicationContext)
            .setApiPublicKey("{your_api_key}") 
            .configure()
    }
}
```

Also `setApiPublicKey()` can be initialized later

```kotlin
sencyMotion.setApiPublicKey("your_api_key") { status ->
    when (status.operation) {
        // handling status
        SencyOperation.UserAuthorization -> {
            when (status.result) {
                is SencyOperationResult.Success<*> ->{}
                is SencyOperationResult.Failure -> {}
            }
        }
    }
}
```

### Usage
The following code must be implemented in an `Activity` or `Fragment` in your application

### Summary result data response
Set up a workout the result listener
```kotlin
val resultLauncher = registerForActivityResult(ActivityResultContracts.StartActivityForResult()) { result ->
    if (result.resultCode == Activity.RESULT_OK) {
        sencyMotion?.onHandleResult(result.data) {
            // get data about the result of workout 
        }
    }
}
```

### Start workout
Start the workout screen. In the parameters method `startWorkout()` add the result listener that you set up above. To track the success of a method or get the expected data, you need to process the result that the methods return in the callback.
```kotlin

// Set workout parameters
val myConfig = WorkoutConfig(
    programId = "{programId}",
    week = 1,
    bodyZone = BodyZone.LowerBody,
    difficultyLevel = DifficultyLevel.HighDifficulty,
    workoutDuration = WorkoutDuration.Long,
    language = SencySupportedLanguage.English
)

sencyMotion.startWorkout(
    launcher = resultLauncher,
    workoutConfig = myConfig
) { status ->
    // handling status
    when (status.operation) {
        SencyOperation.WorkoutInfo -> {
            when (status.result) {
                is SencyOperationResult.Success<*> -> {}
                is SencyOperationResult.Failure -> {}
            }
        }
    }
}
```

### Start custom workout

Start the workout screen with custom workout. The required parameter for starting custom workout it's `CustomWorkout`, 
witch have `introduction`(optional) with type Uri, `soundtrack`(optional) with type Uri and `exercises` 
list of the `SencyWorkout`(required). The `SencyWorkout` has the following implementation 
`Exercise`, `Rest` and `Cooldown`. To the use SDK exercise pass the properties `type` 
in `Exercise` or pass `named` with name of your workout(`type` must be null). \
In the parameters method `startWorkout()` add the result listener that you set up above. 
To track the success of a method or get the expected data, you need to process the 
result that the methods return to the callback.

```kotlin
val exercises = listOf(
    SencyWorkout.Exercise(
        totalSeconds = 10,
        totalReps = 3,
        uiElements = setOf(SencyUiElement.Timer, SencyUiElement.RepsCounter),
        type = SencyExerciseType.Burpees,
        named = "",
    ),
    SencyWorkout.Rest(totalSeconds = 5),
    SencyWorkout.Cooldown(totalSeconds = 5)
)
val workoutAudioVideoFiles = WorkoutAudioVideoFiles(
    introduction = Uri?,
    soundtrack = Uri?,
    phoneCalVideo = Uri?,
    phoneCalAudio = Uri?,
    bodyCalStartAudio = Uri?,
    bodyCalFinishedAudio = Uri?,
)
val customWorkout = CustomWorkout.Builder()
    .setWorkoutAudioVideoFiles(workoutAudioVideoFiles)
    .setExercises(exercises)
    .build()

sencyMotion.startWorkout(
    launcher = resultLauncher,
    customWorkout = customWorkout,
) { status ->
    // handling status
    when (status.operation) {
        SencyOperation.WorkoutInfo -> {
            when (status.result) {
                is SencyOperationResult.Success<*> -> {}
                is SencyOperationResult.Failure -> {}
            }
        }
    }
}
```

### Custom phone moved dialog
Override the default motion dialog for this, inherit your own implementation from the `SencyPhoneMovedDialogFragment` class.
In the parameters method `startWorkout()` add the result listener that you set up above. To track the success of a method or get the expected data, you need to process the result that the methods return in the callback.

```kotlin
class MyMovedDialog : SencyPhoneMovedDialogFragment() {

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        return inflater.inflate(R.layout.my_moved_dialog, container, false)
    }
}
```

React to the display of the dialog by calling the `onClick()` method

* Continue current Workout
```kotlin
onClick(PhoneMovedAction.Resume)
```

* Leave the current Workout
```kotlin
onClick(PhoneMovedAction.QuitWorkout)
```

* Finish current Workout
```kotlin
onClick(PhoneMovedAction.FinishWorkout)
```

Set the new dialog implementation to the parameters of the `startWorkout()` method.
```kotlin
sencyMotion.startWorkout(
    // ... //
    phoneMovedDialogFragment = MyMovedDialog()
) { status ->
    // handling status
    // ... //
}
```

### Get workout info
The `getWorkoutInfo()` method returns a JSON of the workout whose config are expected in the parameters. To track the success of a method or get the expected data, you need to process the result that the methods return in the callback.
```kotlin
// Set workout parameters
val myConfig = WorkoutConfig(
    programId = "{programId}",
    week = 1,
    bodyZone = BodyZone.LowerBody,
    difficultyLevel = DifficultyLevel.HighDifficulty,
    workoutDuration = WorkoutDuration.Long,
    language = SencySupportedLanguage.English
)

sencyMotion.getWorkoutInfo(workoutConfig = myConfig) { result ->
    // handling result and get workout JSON
    when (result.operation) {
        SencyOperation.WorkoutInfo -> {
            when (val jsonResult = result.result) {
                is SencyOperationResult.Success<*> -> {
                    val json = jsonResult.data.toString()
                }
                is SencyOperationResult.Failure -> {}
            }
        }
    }
}
```
