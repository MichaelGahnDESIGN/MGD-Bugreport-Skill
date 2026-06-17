# Godot GDScript – Feedback Hub Integration

Feedback-Hub als Autoload-Singleton plus CanvasLayer-Overlay für den floating Button. Screenshot via `Viewport.get_texture()`.

## Projektstruktur

```
example-app/
├── autoload/
│   └── FeedbackService.gd    ← Autoload-Singleton
├── ui/
│   ├── FeedbackButton.tscn   ← CanvasLayer mit Button
│   └── FeedbackModal.tscn    ← Panel mit Formular
└── project.godot
```

## project.godot – Autoload registrieren

```ini
[autoload]
FeedbackService="*res://autoload/FeedbackService.gd"
```

## FeedbackService Autoload

```gdscript
# autoload/FeedbackService.gd
extends Node

const API_URL = "https://feedback.example.com/api/report"

signal report_submitted(success: bool)

# System-Informationen sammeln
func _get_system_info() -> Dictionary:
    return {
        "os": OS.get_name(),
        "os_version": OS.get_version(),
        "arch": Engine.get_architecture_name() if Engine.has_method("get_architecture_name") else "unknown",
        "app_version": ProjectSettings.get_setting("application/config/version", "1.0.0"),
        "screen_width": DisplayServer.screen_get_size().x,
        "screen_height": DisplayServer.screen_get_size().y,
        "engine_version": Engine.get_version_info().get("string", "unknown"),
    }

# Screenshot vom aktuellen Viewport aufnehmen
# Wichtig: Nur auf Desktop unbedenklich – auf Mobile Nutzer fragen!
func take_screenshot() -> String:
    await RenderingServer.frame_post_draw  # warten bis Frame gerendert ist
    var viewport_texture: ViewportTexture = get_viewport().get_texture()
    var image: Image = viewport_texture.get_image()
    if image == null:
        return ""
    var png_bytes: PackedByteArray = image.save_png_to_buffer()
    return Marshalls.raw_to_base64(png_bytes)

# Bug melden (inkl. Screenshot)
func submit_bug(title: String, description: String,
                severity: String = "medium") -> void:
    var screenshot = await take_screenshot()
    _send_report("bug", title, description, severity, screenshot)

# Idee vorschlagen (kein Screenshot)
func submit_idea(title: String, description: String) -> void:
    _send_report("idea", title, description, "low", "")

# Allgemeines Feedback
func submit_feedback(title: String, description: String) -> void:
    _send_report("feedback", title, description, "low", "")

func _send_report(type: String, title: String, description: String,
                  severity: String, screenshot: String) -> void:
    var body := {
        "type": type,
        "title": title,
        "description": description,
        "severity": severity,
        "screenshot": screenshot if screenshot != "" else null,
        "system_info": _get_system_info(),
        "created_at": Time.get_datetime_string_from_system(true),  # UTC ISO 8601
    }

    var json_string := JSON.stringify(body)
    var headers := PackedStringArray([
        "Content-Type: application/json",
        "Accept: application/json",
    ])

    var http := HTTPRequest.new()
    add_child(http)
    http.request_completed.connect(
        func(result, response_code, _headers, _body):
            var success: bool = result == HTTPRequest.RESULT_SUCCESS \
                                and response_code == 201
            if not success:
                push_warning("Feedback-Fehler: result=%d, code=%d" % [result, response_code])
            report_submitted.emit(success)
            http.queue_free()
    )
    var error := http.request(API_URL, headers, HTTPClient.METHOD_POST, json_string)
    if error != OK:
        push_error("HTTPRequest fehlgeschlagen: %s" % error)
        http.queue_free()
```

## FeedbackButton Scene (CanvasLayer)

```gdscript
# ui/FeedbackButton.gd
# Hängt an einem CanvasLayer mit layer = 99 (immer über dem Spiel-Content)
extends CanvasLayer

@onready var button: Button = $MarginContainer/FeedbackButton
@onready var modal: Control = $FeedbackModal

func _ready() -> void:
    layer = 99  # Hoher Layer-Wert: über allem anderen Content
    button.pressed.connect(_on_button_pressed)
    FeedbackService.report_submitted.connect(_on_report_submitted)

func _on_button_pressed() -> void:
    modal.show()

func _on_report_submitted(success: bool) -> void:
    modal.hide()
    var msg := "Feedback gesendet! Danke." if success \
               else "Fehler – bitte erneut versuchen."
    # Zeige kurze Meldung (z.B. über ein Label oder Toast)
    _show_toast(msg)

func _show_toast(text: String) -> void:
    var label := Label.new()
    label.text = text
    add_child(label)
    await get_tree().create_timer(3.0).timeout
    label.queue_free()
```

## FeedbackModal Script

```gdscript
# ui/FeedbackModal.gd
extends PanelContainer

@onready var type_option: OptionButton = $VBox/TypeOption
@onready var title_input: LineEdit    = $VBox/TitleInput
@onready var desc_input: TextEdit     = $VBox/DescInput
@onready var send_button: Button      = $VBox/HBox/SendButton
@onready var cancel_button: Button    = $VBox/HBox/CancelButton

func _ready() -> void:
    hide()
    cancel_button.pressed.connect(hide)
    send_button.pressed.connect(_on_send)

    type_option.add_item("Bug melden")
    type_option.add_item("Idee vorschlagen")
    type_option.add_item("Allgemeines Feedback")

func _on_send() -> void:
    var title := title_input.text.strip_edges()
    if title.is_empty():
        return
    var desc := desc_input.text.strip_edges()
    send_button.disabled = true
    send_button.text = "Sende..."

    match type_option.selected:
        0: await FeedbackService.submit_bug(title, desc)
        1: FeedbackService.submit_idea(title, desc)
        2: FeedbackService.submit_feedback(title, desc)

    send_button.disabled = false
    send_button.text = "Senden"
    title_input.clear()
    desc_input.clear()
```

## FeedbackButton.tscn (Szenen-Struktur)

```
CanvasLayer (layer=99)          ← FeedbackButton.gd
├── MarginContainer
│   └── Button ("💬 Feedback")  ← anchors: bottom-right, margin 20px
└── FeedbackModal (PanelContainer, hidden) ← FeedbackModal.gd
    └── VBoxContainer
        ├── Label "Feedback senden"
        ├── OptionButton (type_option)
        ├── LineEdit (title_input)
        ├── TextEdit (desc_input)
        └── HBoxContainer
            ├── Button "Abbrechen" (cancel_button)
            └── Button "Senden" (send_button)
```

## Warnung: Screenshot-Datenschutz auf Mobile

```gdscript
# Vor dem Screenshot auf mobilen Plattformen immer prüfen:
func take_screenshot_safely() -> String:
    if OS.get_name() in ["Android", "iOS"]:
        # Nutzer-Bestätigung anfordern, bevor Screenshot gemacht wird
        var confirmed = await _ask_user_permission()
        if not confirmed:
            return ""
    return await take_screenshot()
```

Auf Android und iOS gelten besondere Datenschutzregeln. Nutzer müssen explizit zustimmen, bevor ein Screenshot des Bildschirms an externe Server gesendet wird.
