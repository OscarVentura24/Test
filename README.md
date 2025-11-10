%% Diagrama de clases simplificado
classDiagram
    class GameManager {
        +Instance: GameManager
        +LoadLevel(levelId)
        +UnloadLevel()
        +CurrentLevelData
    }
    class LevelManager {
        +LoadLevelScene(sceneName)
        +OnLevelLoaded()
    }
    class LevelData {
        +string levelId
        +string sceneName
        +Sprite thumbnail
        +int requiredKeys
    }
    class SpawnPoint {
        +enum SpawnType { Player, Key, Door }
        +SpawnType type
    }
    class Player { +Teleport(Vector3 pos) }
    class GameEvents {
        <<static>>
        +event Action<string> OnKeyAcquired
        +TriggerKeyAcquired(string id)
    }
    class CollectibleBase { +Collect() }
    class KeyItem { +itemId }
    class RequirementBase { +IsMet : bool; +Initialize(); +Cleanup(); }
    class KeyRequirement { +keyId }

    GameManager "1" --> "1" LevelManager : uses
    GameManager --> LevelData
    LevelManager --> SpawnPoint
    Player --> SpawnPoint
    KeyItem --> GameEvents : TriggerKeyAcquired
    KeyRequirement --> GameEvents : listens OnKeyAcquired
    DoorController --> RequirementBase
    DoorController --> KeyRequirement

%% Diagrama de secuencia básico: cargar nivel y recoger llave
sequenceDiagram
    participant UI
    participant GameManager
    participant LevelManager
    participant Scene as LevelScene
    participant Player
    participant KeyItem
    participant GameEvents
    participant DoorController

    UI->>GameManager: LoadLevel(levelId)
    GameManager->>LevelManager: LoadLevelScene(sceneName)
    LevelManager->>Scene: LoadSceneAsync(additive)
    Scene->>LevelManager: SceneLoaded
    LevelManager->>Player: TeleportTo(spawnPoint)
    Player->>KeyItem: collide -> Collect()
    KeyItem->>GameEvents: TriggerKeyAcquired(keyId)
    GameEvents->>KeyRequirement: OnKeyAcquired(keyId)
    KeyRequirement->>GameEvents: TriggerRequirementMet(requirementId)
    GameEvents->>DoorController: OnDoorRequirementMet(requirementId)
    DoorController->>DoorController: Unlock & Open
