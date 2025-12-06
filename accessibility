from drafter import *
from dataclasses import dataclass
import textwrap3
import re
from gtts import gTTS
import os
import pygame
from deep_translator import GoogleTranslator

@dataclass
class State:
    blind:bool
    deaf:bool
    language_spoken:str
    language_wanted:str
    colorblind:bool
    dyslexic:bool
    poor_vision:bool
    mobility_issues:bool
    allergies:bool
    keyboard_text:str


def translate(words:str,end:str)->str:
    """
    Helper function for the translation route.
    """
    translated=GoogleTranslator(source='auto', target=end).translate(words)
    return translated



def text_to_speach(words:str)->str:
    if words:
        tts = gTTS(text=words, lang='en', slow=False)
        pygame.mixer.init()
        audio_file = "output.mp3"
        tts.save(audio_file)
        sound = pygame.mixer.Sound(audio_file)
        channel = sound.play()
        os.remove(audio_file)
    return 'done'


def make_dyslexia_friendly(text: str) -> str:
    """
    Applies simple dyslexia-friendly formatting:
    - breaks long lines
    - increases spacing
    - bolds first word of each sentence
    - adds spacing between paragraphs
    
    Args: text (str) - The original text
    Returns: str - The translated text
    """

    sentences = re.split(r'(?<=[.!?]) +', text.strip())

    new_sentences = []
    for s in sentences:
        if not s.strip():
            continue
        parts = s.split(" ", 1)
        if len(parts) > 1:
            s = f"*{parts[0]}* " + parts[1]
        else:
            s = f"*{s}*"

        words = s.split()
        grouped = []
        for i in range(0, len(words), 8):
            grouped.append(" ".join(words[i:i+8]))
        s = "\n".join(grouped)

        new_sentences.append(s)

    return "\n\n".join(new_sentences)

def make_large_print(text: str) -> str:
    """
    Simple formatting aimed at improving readability for people with poor vision:
    - Shorter line lengths
    - Extra spacing between lines
    - capatilized headings if detected
    - Double-spacing between paragraphs
    - Inserts line breaks before long sentences
    
    Args: text (str) - The original text
    Returns: str - The transformed text
    """

    cleaned = text.strip()

    wrapped = textwrap3.fill(cleaned, width=45)

    paragraphs = wrapped.split("\n")
    spaced = "\n\n".join(paragraphs)

    final_lines = []
    sentences = spaced.replace("\n", " ").split(". ")
    for i, s in enumerate(sentences):
        final_lines.append(s.strip())
        if i % 2 == 1:  
            final_lines.append("")

    result = "\n".join(final_lines)

    return result.strip()

@route
def index(state:State) -> Page:
    #text_to_speach('If you are blind please get an aid to assist with program start.')
    return Page(state = state, content = [
        Text("""Welcome to the all in one disability and accesibilty website!
                If you are blind or have extremely poor vision, ask an aid to fill in
                the first part of the website."""),
        Button(text = "Begin", url = "/start")
        ])

@route
def start(state: State) -> Page:
    return Page(
        state=state,
        content=[
            Text("Click each box that applies to you:"),
            Row(CheckBox(name="blind"), Text("Blind / Low Vision")),
            Row(CheckBox(name="deaf"), Text("Deaf / Hard of Hearing")),
            Row(CheckBox(name="colorblind"), Text("Colorblindness")),
            Row(CheckBox(name="dyslexic"), Text("Dyslexia")),
            Row(CheckBox(name="poor_vision"), Text("Poor Vision (not fully blind)")),
            Row(CheckBox(name="mobility_issues"), Text("Mobility / Motor Difficulties")),
            Button(
                text="Click here for assistance with creating a disability, medication, or allergy card",
                url="/card_maker"),
            Button(
                text="Click here for translating services",
                url="/translation"),
            Button(text = "Next", url = "/options")
        ]
    )

@route
def options(state:State, blind:bool, deaf:bool, colorblind:bool, dyslexic:bool, poor_vision:bool, mobility_issues:bool)->Page:
    state.blind = blind
    state.deaf = deaf
    state.colorblind = colorblind
    state.dyslexic = dyslexic
    state.poor_vision = poor_vision
    state.mobility_issues = mobility_issues
    content = ["What would you like help with today?"]
    if state.blind:
        content.append(Button(text = "Text to voice (blind)", url = "/text_to_voice"))
    if state.deaf:
        content.append(Button(text = "Autocaptioning (deaf)", url = "/auto_caption"))
    if state.colorblind:
        content.append(Button(text = "Color deciphering (colorblind)", url = "/colors"))
    if state.dyslexic:
        content.append(Button(text = "Text readability help (dyslexic)", url = "/dyslexia_text"))
    if state.poor_vision:
        content.append(Button(text = "Text readability help (poor vision)", url = "/poor_vision_text"))
    if state.mobility_issues:
        content.append(Button(text = "Button keyboard(mobility limitations)", url = "/keyboard"))
    return Page(state = state, content = content)

@route
def text_to_voice(state:State)->Page:
    return Page(state = state, content = [
        Text("Copy and paste text to be read aloud"),
        TextArea(name = "text", default_value = "Paste here"),
        Button(text = "Read", url = "/read_aloud"),
        Button(text = "Home", url = "/start")])

@route
def read_aloud(state:State, text:str) -> Page:
    text_to_speach(text)
    return text_to_voice(state)

@route
def auto_caption(state:State)-> Page:
    return Page(state = state, content = [
        Text("Upload a video to be captioned"),
        FileUpload(name = "file", accept = ["MP4", "MOV", "WMV", "MKV"]),
        Button(text = "Caption", url = "/caption"),
        Button(text = "Back", url = "/text_to_voice")])

@route
def caption(state: State, file: str) -> Page:
    pass


@route
def colors(state:State) -> Page:
    return Page(state = state, content = [
        Text("Upload a picture to be color analyzed"),
        FileUpload(name = "file"),
        Button(text = "Analyze", url = "/color_analyze"),
        Button(text = "Home", url = "/start")])

@route
def color_analyze(state:State, file:str)->Page:
    pass

@route
def dyslexia_text(state:State)->Page:
    return Page(state = state, content = [
        Text("Copy and paste text to be put into a more readable format"),
        TextArea(name = "text", default_value = "Paste here"),
        Button(text = "Next", url = "/dyslexia_adaptation"),
        Button(text = "Home", url = "/start")])

@route
def dyslexia_adaptation(state: State, text: str) -> Page:
    """
    Takes user-pasted text and returns a dyslexia-friendly version.
    """

    if not text or text.strip().lower() == "paste here":
        adapted = "No text was provided."
    else:
        adapted = make_dyslexia_friendly(text)

    return Page(
        state=state,
        content=[
            Text("Dyslexia-Friendly Output"),
            TextArea(name="adapted_text", default_value=adapted, rows=12),
            Button(text="Back", url="/dyslexia_text")
        ]
    )

@route
def poor_vision_text(state:State)->Page:
    return Page(state = state, content = [
        Text("Copy and paste text to be put into a more readable format"),
        TextArea(name = "text", default_value = "Paste here"),
        Button(text = "Next", url = "/poor_vision_adaptation")])



@route
def poor_vision_adaptation(state: State, text: str) -> Page:
    """
    Takes user-pasted text and returns a large-print,
    high-visibility version for people with poor vision.
    """

    if not text or text.strip().lower() == "paste here":
        adapted = "No text was provided."
    else:
        adapted = make_large_print(text)

    return Page(
        state=state,
        content=[
            Text("Large-Print / High-Visibility Output"),
            TextArea(
                name="adapted_text",
                default_value=adapted,
                rows=12,
            ),
            Button(text="Back", url="/poor_vision_text")
        ]
    )

@route
def press_key(state: State, key: str) -> Page:
    if not hasattr(state, "keyboard_text"):
        state.keyboard_text = ""

    if key == "_space_":
        state.keyboard_text += " "
    elif key == "_back_":
        state.keyboard_text = state.keyboard_text[:-1]
    elif key == "_clear_":
        state.keyboard_text = ""
    else:
        state.keyboard_text += key

    return keyboard(state)


@route
def keyboard(state: State) -> Page:
    if not hasattr(state, "keyboard_text"):
        state.keyboard_text = ""

    row1 = "Q W E R T Y U I O P".split()
    row2 = "A S D F G H J K L".split()
    row3 = "Z X C V B N M".split()

    def make_row(letters):
        return Row(*[
            Button(
                text=letter,
                url=f"/press_key?key={letter}",
                font_size="24px",
                padding="16px",
                width="60px"
            )
            for letter in letters
        ])

    return Page(
        state=state,
        content=[
            Text("Accessible On-Screen Keyboard"),
            TextArea(
                name="keyboard_text",
                default_value=state.keyboard_text,
                rows=4,
                font_size="20px"
            ),

            make_row(row1),
            make_row(row2),
            make_row(row3),

            Row(
                Button(
                    text="SPACE",
                    url="/press_key?key=_space_",
                    font_size="20px",
                    padding="16px",
                    width="150px"
                ),
                Button(
                    text="BACKSPACE",
                    url="/press_key?key=_back_",
                    font_size="20px",
                    padding="16px",
                    width="150px"
                ),
                Button(
                    text="CLEAR",
                    url="/press_key?key=_clear_",
                    font_size="20px",
                    padding="16px",
                    width="120px"
                ),
                Button(text = "Home", url = "/start")
            )
        ]
    )

@route
def card_maker(state:State)->Page:
    return Page(state = state, content = [
        Text("""Many people with non-visible illnesses or disabilities carry
                carry a card containing vital medical information for
                emergencies. Please fill in the information to have your own
                card designed:"""),
        TextBox(name = "name", default_value = "Name"),
        TextBox(name = "emergency_contact", default_value = "Emergency Contact"),
        TextBox(name = "allergies", default_value = "Allergies"),
        TextBox(name = "conditions", default_value = "Known medical conditions"),
        Button(text = "Generate Card", url = "/card_generator")])

@route
def card_generator(state: State, name: str, emergency_contact: str, allergies: str, conditions: str) -> Page:
    card_content = [
        Text("💳 Your Emergency Medical Card 💳"),
        Text(f"Name: {name}"),
        Text(f"Emergency Contact: {emergency_contact}"),
        Text(f"Allergies: {allergies}"),
        Text(f"Medical Conditions: {conditions}"),
        Text("Keep this card with you at all times."),
        Button(text="Create Another Card", url="/card_maker"),
        Button(text = "Home", url = "/start")
    ]

    return Page(state=state, content=card_content)
@route
def translation(state:State)->Page:
    return Page(state,
                content=[Text('Enter text  to translate into'),
                         Text('common languages: English-en, Spanish-es, German-de, French-fr, Italian-it'),
                         Row(TextBox('text','Enter Text Here'),
                             TextBox('key','Enter 2 Letter Key')),
                         Button('Translate','/translation2'),
                         Button('Go Home','/start')])
@route
def translation2(state:State,key:str,text:str)->Page:
    translated=translate(text,key)
    return Page(state,
                content=[Text(translated),
                         Button('Return','/translation')])


start_server(State(False, False, "", "", False, False, False, False, False, ""))
    
