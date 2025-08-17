+++
title = 'Emojis pour Jekyll via Jemoji'
date = 2025-03-18 00:00:00 +0100
categories = markdown
+++
Cette aide-mémoire répertorie tous les émojis disponibles et fonctionnels , utilisables dans les documents Markdown des sites web créés avec Jekyll . 


Date: {{ page.date | date: "%d/%m/%Y" }}  
Date `last_modified_at`: {{ page.last_modified_at | date: "%d/%m/%Y" }}  
Date et heure actuelle: {{ "now" | date: "%d/%m/%Y %H:%M" }}

## Activer Jemoji sur votre site 

Ajoutez l'entrée suivante à votre Gemlile:

```
gem 'jemoji'
```

Ajoutez également l'entrée suivante à la liste des plugins dans votre `_config.yml`:

```
plugins:
  - jemoji
```

Mettez à jour le bundle pour votre build local avec la commande de terminal suivante :

```
bundle update
```

## Emoji

### Objets

#### Ordinateur

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :battery: | `:battery:` | :electric_plug: | `:electric_plug:` |
| :computer: | `:computer:` | :desktop_computer: | `:desktop_computer:` |
| :printer: | `:printer:` | :keyboard: | `:keyboard:` |
| :computer_mouse: | `:computer_mouse:` | :trackball: | `:trackball:` |
| :minidisc: | `:minidisc:` | :floppy_disk: | `:floppy_disk:` |
| :cd: | `:cd:` | :dvd: | `:dvd:` |
| :abacus: | `:abacus:` | | |

#### Courriel

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :envelope: | `:envelope:` | :e-mail: | `:e-mail:` <br /> `:email:` |
| :incoming_envelope: | `:incoming_envelope:` | :envelope_with_arrow: | `:envelope_with_arrow:` |
| :outbox_tray: | `:outbox_tray:` | :inbox_tray: | `:inbox_tray:` |
| :package: | `:package:` | :mailbox: | `:mailbox:` |
| :mailbox_closed: | `:mailbox_closed:` | :mailbox_with_mail: | `:mailbox_with_mail:` |
| :mailbox_with_no_mail: | `:mailbox_with_no_mail:` | :postbox: | `:postbox:` |
| :ballot_box: | `:ballot_box:` | | |

#### Bureau

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :briefcase: | `:briefcase:` | :file_folder: | `:file_folder:` |
| :open_file_folder: | `:open_file_folder:` | :card_index_dividers: | `:card_index_dividers:` |
| :date: | `:date:` | :calendar: | `:calendar:` |
| :spiral_notepad: | `:spiral_notepad:` | :spiral_calendar: | `:spiral_calendar:` |
| :card_index: | `:card_index:` | :chart_with_upwards_trend: | `:chart_with_upwards_trend:` |
| :chart_with_downwards_trend: | `:chart_with_downwards_trend:` | :bar_chart: | `:bar_chart:` |
| :clipboard: | `:clipboard:` | :pushpin: | `:pushpin:` |
| :round_pushpin: | `:round_pushpin:` | :paperclip: | `:paperclip:` |
| :paperclips: | `:paperclips:` | :straight_ruler: | `:straight_ruler:` |
| :triangular_ruler: | `:triangular_ruler:` | :scissors: | `:scissors:` |
| :card_file_box: | `:card_file_box:` | :file_cabinet: | `:file_cabinet:` |
| :wastebasket: | `:wastebasket:` | | |

#### Verrouillage

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :lock: | `:lock:` | :unlock: | `:unlock:` |
| :lock_with_ink_pen: | `:lock_with_ink_pen:` | :closed_lock_with_key: | `:closed_lock_with_key:` |
| :key: | `:key:` | :old_key: | `:old_key:` |

#### Outil

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :hammer: | `:hammer:` | :axe: | `:axe:` |
| :pick: | `:pick:` | :hammer_and_pick: | `:hammer_and_pick:` |
| :hammer_and_wrench: | `:hammer_and_wrench:` | :dagger: | `:dagger:` |
| :crossed_swords: | `:crossed_swords:` | :bomb: | `:bomb:` |
| :boomerang: | `:boomerang:` | :bow_and_arrow: | `:bow_and_arrow:` |
| :shield: | `:shield:` | :carpentry_saw: | `:carpentry_saw:` |
| :wrench: | `:wrench:` | :screwdriver: | `:screwdriver:` |
| :nut_and_bolt: | `:nut_and_bolt:` | :gear: | `:gear:` |
| :clamp: | `:clamp:` | :balance_scale: | `:balance_scale:` |
| :probing_cane: | `:probing_cane:` | :link: | `:link:` |
| :chains: | `:chains:` | :hook: | `:hook:` |
| :toolbox: | `:toolbox:` | :magnet: | `:magnet:` |
| :ladder: | `:ladder:` | | |

#### Vêtements

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :eyeglasses: | `:eyeglasses:` | :dark_sunglasses: | `:dark_sunglasses:` |
| :goggles: | `:goggles:` | :lab_coat: | `:lab_coat:` |
| :safety_vest: | `:safety_vest:` | :necktie: | `:necktie:` |
| :shirt: | `:shirt:` <br /> `:tshirt:` | :jeans: | `:jeans:` |
| :scarf: | `:scarf:` | :gloves: | `:gloves:` |
| :coat: | `:coat:` | :socks: | `:socks:` |
| :dress: | `:dress:` | :kimono: | `:kimono:` |
| :sari: | `:sari:` | :one_piece_swimsuit: | `:one_piece_swimsuit:` |
| :swim_brief: | `:swim_brief:` | :shorts: | `:shorts:` |
| :bikini: | `:bikini:` | :womans_clothes: | `:womans_clothes:` |
| :purse: | `:purse:` | :handbag: | `:handbag:` |
| :pouch: | `:pouch:` | :shopping: | `:shopping:` |
| :school_satchel: | `:school_satchel:` | :thong_sandal: | `:thong_sandal:` |
| :mans_shoe: | `:mans_shoe:` <br /> `:shoe:` | :athletic_shoe: | `:athletic_shoe:` |
| :hiking_boot: | `:hiking_boot:` | :flat_shoe: | `:flat_shoe:` |
| :high_heel: | `:high_heel:` | :sandal: | `:sandal:` |
| :ballet_shoes: | `:ballet_shoes:` | :boot: | `:boot:` |
| :crown: | `:crown:` | :womans_hat: | `:womans_hat:` |
| :tophat: | `:tophat:` | :mortar_board: | `:mortar_board:` |
| :billed_cap: | `:billed_cap:` | :military_helmet: | `:military_helmet:` |
| :rescue_worker_helmet: | `:rescue_worker_helmet:` | :prayer_beads: | `:prayer_beads:` |
| :lipstick: | `:lipstick:` | :ring: | `:ring:` |
| :gem: | `:gem:` | | |

#### Son

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :mute: | `:mute:` | :speaker: | `:speaker:` |
| :sound: | `:sound:` | :loud_sound: | `:loud_sound:` |
| :loudspeaker: | `:loudspeaker:` | :mega: | `:mega:` |
| :postal_horn: | `:postal_horn:` | :bell: | `:bell:` |
| :no_bell: | `:no_bell:` | | |

#### Musique

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :musical_score: | `:musical_score:` | :musical_note: | `:musical_note:` |
| :notes: | `:notes:` | :studio_microphone: | `:studio_microphone:` |
| :level_slider: | `:level_slider:` | :control_knobs: | `:control_knobs:` |
| :microphone: | `:microphone:` | :headphones: | `:headphones:` |
| :radio: | `:radio:` | | |

#### Instrument de musique

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :saxophone: | `:saxophone:` | :accordion: | `:accordion:` |
| :guitar: | `:guitar:` | :musical_keyboard: | `:musical_keyboard:` |
| :trumpet: | `:trumpet:` | :violin: | `:violin:` |
| :banjo: | `:banjo:` | :drum: | `:drum:` |
| :long_drum: | `:long_drum:` | | |

#### Téléphone

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :iphone: | `:iphone:` | :calling: | `:calling:` |
| :phone: | `:phone:` <br /> `:telephone:` | :telephone_receiver: | `:telephone_receiver:` |
| :pager: | `:pager:` | :fax: | `:fax:` |

#### Lumière et vidéo

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :movie_camera: | `:movie_camera:` | :film_strip: | `:film_strip:` |
| :film_projector: | `:film_projector:` | :clapper: | `:clapper:` |
| :tv: | `:tv:` | :camera: | `:camera:` |
| :camera_flash: | `:camera_flash:` | :video_camera: | `:video_camera:` |
| :vhs: | `:vhs:` | :mag: | `:mag:` |
| :mag_right: | `:mag_right:` | :candle: | `:candle:` |
| :bulb: | `:bulb:` | :flashlight: | `:flashlight:` |
| :izakaya_lantern: | `:izakaya_lantern:` <br /> `:lantern:` | :diya_lamp: | `:diya_lamp:` |

#### Ouvrages

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :notebook_with_decorative_cover: | `:notebook_with_decorative_cover:` | :closed_book: | `:closed_book:` |
| :book: | `:book:` <br /> `:open_book:` | :green_book: | `:green_book:` |
| :blue_book: | `:blue_book:` | :orange_book: | `:orange_book:` |
| :books: | `:books:` | :notebook: | `:notebook:` |
| :ledger: | `:ledger:` | :page_with_curl: | `:page_with_curl:` |
| :scroll: | `:scroll:` | :page_facing_up: | `:page_facing_up:` |
| :newspaper: | `:newspaper:` | :newspaper_roll: | `:newspaper_roll:` |
| :bookmark_tabs: | `:bookmark_tabs:` | :bookmark: | `:bookmark:` |
| :label: | `:label:` | | |

#### Monnaie

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :moneybag: | `:moneybag:` | :coin: | `:coin:` |
| :yen: | `:yen:` | :dollar: | `:dollar:` |
| :euro: | `:euro:` | :pound: | `:pound:` |
| :money_with_wings: | `:money_with_wings:` | :credit_card: | `:credit_card:` |
| :receipt: | `:receipt:` | :chart: | `:chart:` |

#### Écriture

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :pencil2: | `:pencil2:` | :black_nib: | `:black_nib:` |
| :fountain_pen: | `:fountain_pen:` | :pen: | `:pen:` |
| :paintbrush: | `:paintbrush:` | :crayon: | `:crayon:` |
| :memo: | `:memo:` <br /> `:pencil:` | | |

#### Science

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :alembic: | `:alembic:` | :test_tube: | `:test_tube:` |
| :petri_dish: | `:petri_dish:` | :dna: | `:dna:` |
| :microscope: | `:microscope:` | :telescope: | `:telescope:` |
| :satellite: | `:satellite:` | | |

#### Medical

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :syringe: | `:syringe:` | :drop_of_blood: | `:drop_of_blood:` |
| :pill: | `:pill:` | :adhesive_bandage: | `:adhesive_bandage:` |
| :stethoscope: | `:stethoscope:` | | |

#### Ménage

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :door: | `:door:` | :elevator: | `:elevator:` |
| :mirror: | `:mirror:` | :window: | `:window:` |
| :bed: | `:bed:` | :couch_and_lamp: | `:couch_and_lamp:` |
| :chair: | `:chair:` | :toilet: | `:toilet:` |
| :plunger: | `:plunger:` | :shower: | `:shower:` |
| :bathtub: | `:bathtub:` | :mouse_trap: | `:mouse_trap:` |
| :razor: | `:razor:` | :lotion_bottle: | `:lotion_bottle:` |
| :safety_pin: | `:safety_pin:` | :broom: | `:broom:` |
| :basket: | `:basket:` | :roll_of_paper: | `:roll_of_paper:` |
| :bucket: | `:bucket:` | :soap: | `:soap:` |
| :toothbrush: | `:toothbrush:` | :sponge: | `:sponge:` |
| :fire_extinguisher: | `:fire_extinguisher:` | :shopping_cart: | `:shopping_cart:` |

#### Autre objet

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :smoking: | `:smoking:` | :coffin: | `:coffin:` |
| :headstone: | `:headstone:` | :funeral_urn: | `:funeral_urn:` |
| :nazar_amulet: | `:nazar_amulet:` | :moyai: | `:moyai:` |
| :placard: | `:placard:` | | |

### Symboles

#### Panneau de transport

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#symbols) | :atm: | `:atm:` | :put_litter_in_its_place: | `:put_litter_in_its_place:` |
| [top](#symbols) | :potable_water: | `:potable_water:` | :wheelchair: | `:wheelchair:` |
| [top](#symbols) | :mens: | `:mens:` | :womens: | `:womens:` |
| [top](#symbols) | :restroom: | `:restroom:` | :baby_symbol: | `:baby_symbol:` |
| [top](#symbols) | :wc: | `:wc:` | :passport_control: | `:passport_control:` |
| [top](#symbols) | :customs: | `:customs:` | :baggage_claim: | `:baggage_claim:` |
| [top](#symbols) | :left_luggage: | `:left_luggage:` | | |

#### Avertissement

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#symbols) | :warning: | `:warning:` | :children_crossing: | `:children_crossing:` |
| [top](#symbols) | :no_entry: | `:no_entry:` | :no_entry_sign: | `:no_entry_sign:` |
| [top](#symbols) | :no_bicycles: | `:no_bicycles:` | :no_smoking: | `:no_smoking:` |
| [top](#symbols) | :do_not_litter: | `:do_not_litter:` | :non-potable_water: | `:non-potable_water:` |
| [top](#symbols) | :no_pedestrians: | `:no_pedestrians:` | :no_mobile_phones: | `:no_mobile_phones:` |
| [top](#symbols) | :underage: | `:underage:` | :radioactive: | `:radioactive:` |
| [top](#symbols) | :biohazard: | `:biohazard:` | | |

#### Flèche

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#symbols) | :arrow_up: | `:arrow_up:` | :arrow_upper_right: | `:arrow_upper_right:` |
| [top](#symbols) | :arrow_right: | `:arrow_right:` | :arrow_lower_right: | `:arrow_lower_right:` |
| [top](#symbols) | :arrow_down: | `:arrow_down:` | :arrow_lower_left: | `:arrow_lower_left:` |
| [top](#symbols) | :arrow_left: | `:arrow_left:` | :arrow_upper_left: | `:arrow_upper_left:` |
| [top](#symbols) | :arrow_up_down: | `:arrow_up_down:` | :left_right_arrow: | `:left_right_arrow:` |
| [top](#symbols) | :leftwards_arrow_with_hook: | `:leftwards_arrow_with_hook:` | :arrow_right_hook: | `:arrow_right_hook:` |
| [top](#symbols) | :arrow_heading_up: | `:arrow_heading_up:` | :arrow_heading_down: | `:arrow_heading_down:` |
| [top](#symbols) | :arrows_clockwise: | `:arrows_clockwise:` | :arrows_counterclockwise: | `:arrows_counterclockwise:` |
| [top](#symbols) | :back: | `:back:` | :end: | `:end:` |
| [top](#symbols) | :on: | `:on:` | :soon: | `:soon:` |
| [top](#symbols) | :top: | `:top:` | | |

#### Zodiac

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#symbols) | :aries: | `:aries:` | :taurus: | `:taurus:` |
| [top](#symbols) | :gemini: | `:gemini:` | :cancer: | `:cancer:` |
| [top](#symbols) | :leo: | `:leo:` | :virgo: | `:virgo:` |
| [top](#symbols) | :libra: | `:libra:` | :scorpius: | `:scorpius:` |
| [top](#symbols) | :sagittarius: | `:sagittarius:` | :capricorn: | `:capricorn:` |
| [top](#symbols) | :aquarius: | `:aquarius:` | :pisces: | `:pisces:` |
| [top](#symbols) | :ophiuchus: | `:ophiuchus:` | | |

#### Symbole Av

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#symbols) | :twisted_rightwards_arrows: | `:twisted_rightwards_arrows:` | :repeat: | `:repeat:` |
| [top](#symbols) | :repeat_one: | `:repeat_one:` | :arrow_forward: | `:arrow_forward:` |
| [top](#symbols) | :fast_forward: | `:fast_forward:` | :next_track_button: | `:next_track_button:` |
| [top](#symbols) | :play_or_pause_button: | `:play_or_pause_button:` | :arrow_backward: | `:arrow_backward:` |
| [top](#symbols) | :rewind: | `:rewind:` | :previous_track_button: | `:previous_track_button:` |
| [top](#symbols) | :arrow_up_small: | `:arrow_up_small:` | :arrow_double_up: | `:arrow_double_up:` |
| [top](#symbols) | :arrow_down_small: | `:arrow_down_small:` | :arrow_double_down: | `:arrow_double_down:` |
| [top](#symbols) | :pause_button: | `:pause_button:` | :stop_button: | `:stop_button:` |
| [top](#symbols) | :record_button: | `:record_button:` | :eject_button: | `:eject_button:` |
| [top](#symbols) | :cinema: | `:cinema:` | :low_brightness: | `:low_brightness:` |
| [top](#symbols) | :high_brightness: | `:high_brightness:` | :signal_strength: | `:signal_strength:` |
| [top](#symbols) | :vibration_mode: | `:vibration_mode:` | :mobile_phone_off: | `:mobile_phone_off:` |

#### Genre

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#symbols) | :female_sign: | `:female_sign:` | :male_sign: | `:male_sign:` |
| [top](#symbols) | :transgender_symbol: | `:transgender_symbol:` | | |

#### Math

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#symbols) | :heavy_multiplication_x: | `:heavy_multiplication_x:` | :heavy_plus_sign: | `:heavy_plus_sign:` |
| [top](#symbols) | :heavy_minus_sign: | `:heavy_minus_sign:` | :heavy_division_sign: | `:heavy_division_sign:` |
| [top](#symbols) | :infinity: | `:infinity:` | | |

#### Ponctuation

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#symbols) | :bangbang: | `:bangbang:` | :interrobang: | `:interrobang:` |
| [top](#symbols) | :question: | `:question:` | :grey_question: | `:grey_question:` |
| [top](#symbols) | :grey_exclamation: | `:grey_exclamation:` | :exclamation: | `:exclamation:` <br /> `:heavy_exclamation_mark:` |
| [top](#symbols) | :wavy_dash: | `:wavy_dash:` | | |

#### Monnaie

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#symbols) | :currency_exchange: | `:currency_exchange:` | :heavy_dollar_sign: | `:heavy_dollar_sign:` |

#### Autre symbole

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#symbols) | :medical_symbol: | `:medical_symbol:` | :recycle: | `:recycle:` |
| [top](#symbols) | :fleur_de_lis: | `:fleur_de_lis:` | :trident: | `:trident:` |
| [top](#symbols) | :name_badge: | `:name_badge:` | :beginner: | `:beginner:` |
| [top](#symbols) | :o: | `:o:` | :white_check_mark: | `:white_check_mark:` |
| [top](#symbols) | :ballot_box_with_check: | `:ballot_box_with_check:` | :heavy_check_mark: | `:heavy_check_mark:` |
| [top](#symbols) | :x: | `:x:` | :negative_squared_cross_mark: | `:negative_squared_cross_mark:` |
| [top](#symbols) | :curly_loop: | `:curly_loop:` | :loop: | `:loop:` |
| [top](#symbols) | :part_alternation_mark: | `:part_alternation_mark:` | :eight_spoked_asterisk: | `:eight_spoked_asterisk:` |
| [top](#symbols) | :eight_pointed_black_star: | `:eight_pointed_black_star:` | :sparkle: | `:sparkle:` |
| [top](#symbols) | :copyright: | `:copyright:` | :registered: | `:registered:` |
| [top](#symbols) | :tm: | `:tm:` | | |

#### Keycap

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#symbols) | :hash: | `:hash:` | :asterisk: | `:asterisk:` |
| [top](#symbols) | :zero: | `:zero:` | :one: | `:one:` |
| [top](#symbols) | :two: | `:two:` | :three: | `:three:` |
| [top](#symbols) | :four: | `:four:` | :five: | `:five:` |
| [top](#symbols) | :six: | `:six:` | :seven: | `:seven:` |
| [top](#symbols) | :eight: | `:eight:` | :nine: | `:nine:` |
| [top](#symbols) | :keycap_ten: | `:keycap_ten:` | | |

#### Alphanum

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#symbols) | :capital_abcd: | `:capital_abcd:` | :abcd: | `:abcd:` |
| [top](#symbols) | :1234: | `:1234:` | :symbols: | `:symbols:` |
| [top](#symbols) | :abc: | `:abc:` | :a: | `:a:` |
| [top](#symbols) | :ab: | `:ab:` | :b: | `:b:` |
| [top](#symbols) | :cl: | `:cl:` | :cool: | `:cool:` |
| [top](#symbols) | :free: | `:free:` | :information_source: | `:information_source:` |
| [top](#symbols) | :id: | `:id:` | :m: | `:m:` |
| [top](#symbols) | :new: | `:new:` | :ng: | `:ng:` |
| [top](#symbols) | :o2: | `:o2:` | :ok: | `:ok:` |
| [top](#symbols) | :parking: | `:parking:` | :sos: | `:sos:` |
| [top](#symbols) | :up: | `:up:` | :vs: | `:vs:` |
| [top](#symbols) | :koko: | `:koko:` | :sa: | `:sa:` |
| [top](#symbols) | :u6708: | `:u6708:` | :u6709: | `:u6709:` |
| [top](#symbols) | :u6307: | `:u6307:` | :ideograph_advantage: | `:ideograph_advantage:` |
| [top](#symbols) | :u5272: | `:u5272:` | :u7121: | `:u7121:` |
| [top](#symbols) | :u7981: | `:u7981:` | :accept: | `:accept:` |
| [top](#symbols) | :u7533: | `:u7533:` | :u5408: | `:u5408:` |
| [top](#symbols) | :u7a7a: | `:u7a7a:` | :congratulations: | `:congratulations:` |
| [top](#symbols) | :secret: | `:secret:` | :u55b6: | `:u55b6:` |
| [top](#symbols) | :u6e80: | `:u6e80:` | | |

#### Geometrique

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#symbols) | :red_circle: | `:red_circle:` | :orange_circle: | `:orange_circle:` |
| [top](#symbols) | :yellow_circle: | `:yellow_circle:` | :green_circle: | `:green_circle:` |
| [top](#symbols) | :large_blue_circle: | `:large_blue_circle:` | :purple_circle: | `:purple_circle:` |
| [top](#symbols) | :brown_circle: | `:brown_circle:` | :black_circle: | `:black_circle:` |
| [top](#symbols) | :white_circle: | `:white_circle:` | :red_square: | `:red_square:` |
| [top](#symbols) | :orange_square: | `:orange_square:` | :yellow_square: | `:yellow_square:` |
| [top](#symbols) | :green_square: | `:green_square:` | :blue_square: | `:blue_square:` |
| [top](#symbols) | :purple_square: | `:purple_square:` | :brown_square: | `:brown_square:` |
| [top](#symbols) | :black_large_square: | `:black_large_square:` | :white_large_square: | `:white_large_square:` |
| [top](#symbols) | :black_medium_square: | `:black_medium_square:` | :white_medium_square: | `:white_medium_square:` |
| [top](#symbols) | :black_medium_small_square: | `:black_medium_small_square:` | :white_medium_small_square: | `:white_medium_small_square:` |
| [top](#symbols) | :black_small_square: | `:black_small_square:` | :white_small_square: | `:white_small_square:` |
| [top](#symbols) | :large_orange_diamond: | `:large_orange_diamond:` | :large_blue_diamond: | `:large_blue_diamond:` |
| [top](#symbols) | :small_orange_diamond: | `:small_orange_diamond:` | :small_blue_diamond: | `:small_blue_diamond:` |
| [top](#symbols) | :small_red_triangle: | `:small_red_triangle:` | :small_red_triangle_down: | `:small_red_triangle_down:` |
| [top](#symbols) | :diamond_shape_with_a_dot_inside: | `:diamond_shape_with_a_dot_inside:` | :radio_button: | `:radio_button:` |
| [top](#symbols) | :white_square_button: | `:white_square_button:` | :black_square_button: | `:black_square_button:` |

### Drapeaux

#### Drapeau

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#flags) | :checkered_flag: | `:checkered_flag:` | :triangular_flag_on_post: | `:triangular_flag_on_post:` |
| [top](#flags) | :crossed_flags: | `:crossed_flags:` | :black_flag: | `:black_flag:` |
| [top](#flags) | :white_flag: | `:white_flag:` | :rainbow_flag: | `:rainbow_flag:` |
| [top](#flags) | :transgender_flag: | `:transgender_flag:` | :pirate_flag: | `:pirate_flag:` |

#### Drapeau pays

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#flags) | :ascension_island: | `:ascension_island:` | :andorra: | `:andorra:` |
| [top](#flags) | :united_arab_emirates: | `:united_arab_emirates:` | :afghanistan: | `:afghanistan:` |
| [top](#flags) | :antigua_barbuda: | `:antigua_barbuda:` | :anguilla: | `:anguilla:` |
| [top](#flags) | :albania: | `:albania:` | :armenia: | `:armenia:` |
| [top](#flags) | :angola: | `:angola:` | :antarctica: | `:antarctica:` |
| [top](#flags) | :argentina: | `:argentina:` | :american_samoa: | `:american_samoa:` |
| [top](#flags) | :austria: | `:austria:` | :australia: | `:australia:` |
| [top](#flags) | :aruba: | `:aruba:` | :aland_islands: | `:aland_islands:` |
| [top](#flags) | :azerbaijan: | `:azerbaijan:` | :bosnia_herzegovina: | `:bosnia_herzegovina:` |
| [top](#flags) | :barbados: | `:barbados:` | :bangladesh: | `:bangladesh:` |
| [top](#flags) | :belgium: | `:belgium:` | :burkina_faso: | `:burkina_faso:` |
| [top](#flags) | :bulgaria: | `:bulgaria:` | :bahrain: | `:bahrain:` |
| [top](#flags) | :burundi: | `:burundi:` | :benin: | `:benin:` |
| [top](#flags) | :st_barthelemy: | `:st_barthelemy:` | :bermuda: | `:bermuda:` |
| [top](#flags) | :brunei: | `:brunei:` | :bolivia: | `:bolivia:` |
| [top](#flags) | :caribbean_netherlands: | `:caribbean_netherlands:` | :brazil: | `:brazil:` |
| [top](#flags) | :bahamas: | `:bahamas:` | :bhutan: | `:bhutan:` |
| [top](#flags) | :bouvet_island: | `:bouvet_island:` | :botswana: | `:botswana:` |
| [top](#flags) | :belarus: | `:belarus:` | :belize: | `:belize:` |
| [top](#flags) | :canada: | `:canada:` | :cocos_islands: | `:cocos_islands:` |
| [top](#flags) | :congo_kinshasa: | `:congo_kinshasa:` | :central_african_republic: | `:central_african_republic:` |
| [top](#flags) | :congo_brazzaville: | `:congo_brazzaville:` | :switzerland: | `:switzerland:` |
| [top](#flags) | :cote_divoire: | `:cote_divoire:` | :cook_islands: | `:cook_islands:` |
| [top](#flags) | :chile: | `:chile:` | :cameroon: | `:cameroon:` |
| [top](#flags) | :cn: | `:cn:` | :colombia: | `:colombia:` |
| [top](#flags) | :clipperton_island: | `:clipperton_island:` | :costa_rica: | `:costa_rica:` |
| [top](#flags) | :cuba: | `:cuba:` | :cape_verde: | `:cape_verde:` |
| [top](#flags) | :curacao: | `:curacao:` | :christmas_island: | `:christmas_island:` |
| [top](#flags) | :cyprus: | `:cyprus:` | :czech_republic: | `:czech_republic:` |
| [top](#flags) | :de: | `:de:` | :diego_garcia: | `:diego_garcia:` |
| [top](#flags) | :djibouti: | `:djibouti:` | :denmark: | `:denmark:` |
| [top](#flags) | :dominica: | `:dominica:` | :dominican_republic: | `:dominican_republic:` |
| [top](#flags) | :algeria: | `:algeria:` | :ceuta_melilla: | `:ceuta_melilla:` |
| [top](#flags) | :ecuador: | `:ecuador:` | :estonia: | `:estonia:` |
| [top](#flags) | :egypt: | `:egypt:` | :western_sahara: | `:western_sahara:` |
| [top](#flags) | :eritrea: | `:eritrea:` | :es: | `:es:` |
| [top](#flags) | :ethiopia: | `:ethiopia:` | :eu: | `:eu:` <br /> `:european_union:` |
| [top](#flags) | :finland: | `:finland:` | :fiji: | `:fiji:` |
| [top](#flags) | :falkland_islands: | `:falkland_islands:` | :micronesia: | `:micronesia:` |
| [top](#flags) | :faroe_islands: | `:faroe_islands:` | :fr: | `:fr:` |
| [top](#flags) | :gabon: | `:gabon:` | :gb: | `:gb:` <br /> `:uk:` |
| [top](#flags) | :grenada: | `:grenada:` | :georgia: | `:georgia:` |
| [top](#flags) | :french_guiana: | `:french_guiana:` | :guernsey: | `:guernsey:` |
| [top](#flags) | :ghana: | `:ghana:` | :gibraltar: | `:gibraltar:` |
| [top](#flags) | :greenland: | `:greenland:` | :gambia: | `:gambia:` |
| [top](#flags) | :guinea: | `:guinea:` | :guadeloupe: | `:guadeloupe:` |
| [top](#flags) | :equatorial_guinea: | `:equatorial_guinea:` | :greece: | `:greece:` |
| [top](#flags) | :south_georgia_south_sandwich_islands: | `:south_georgia_south_sandwich_islands:` | :guatemala: | `:guatemala:` |
| [top](#flags) | :guam: | `:guam:` | :guinea_bissau: | `:guinea_bissau:` |
| [top](#flags) | :guyana: | `:guyana:` | :hong_kong: | `:hong_kong:` |
| [top](#flags) | :heard_mcdonald_islands: | `:heard_mcdonald_islands:` | :honduras: | `:honduras:` |
| [top](#flags) | :croatia: | `:croatia:` | :haiti: | `:haiti:` |
| [top](#flags) | :hungary: | `:hungary:` | :canary_islands: | `:canary_islands:` |
| [top](#flags) | :indonesia: | `:indonesia:` | :ireland: | `:ireland:` |
| [top](#flags) | :israel: | `:israel:` | :isle_of_man: | `:isle_of_man:` |
| [top](#flags) | :india: | `:india:` | :british_indian_ocean_territory: | `:british_indian_ocean_territory:` |
| [top](#flags) | :iraq: | `:iraq:` | :iran: | `:iran:` |
| [top](#flags) | :iceland: | `:iceland:` | :it: | `:it:` |
| [top](#flags) | :jersey: | `:jersey:` | :jamaica: | `:jamaica:` |
| [top](#flags) | :jordan: | `:jordan:` | :jp: | `:jp:` |
| [top](#flags) | :kenya: | `:kenya:` | :kyrgyzstan: | `:kyrgyzstan:` |
| [top](#flags) | :cambodia: | `:cambodia:` | :kiribati: | `:kiribati:` |
| [top](#flags) | :comoros: | `:comoros:` | :st_kitts_nevis: | `:st_kitts_nevis:` |
| [top](#flags) | :north_korea: | `:north_korea:` | :kr: | `:kr:` |
| [top](#flags) | :kuwait: | `:kuwait:` | :cayman_islands: | `:cayman_islands:` |
| [top](#flags) | :kazakhstan: | `:kazakhstan:` | :laos: | `:laos:` |
| [top](#flags) | :lebanon: | `:lebanon:` | :st_lucia: | `:st_lucia:` |
| [top](#flags) | :liechtenstein: | `:liechtenstein:` | :sri_lanka: | `:sri_lanka:` |
| [top](#flags) | :liberia: | `:liberia:` | :lesotho: | `:lesotho:` |
| [top](#flags) | :lithuania: | `:lithuania:` | :luxembourg: | `:luxembourg:` |
| [top](#flags) | :latvia: | `:latvia:` | :libya: | `:libya:` |
| [top](#flags) | :morocco: | `:morocco:` | :monaco: | `:monaco:` |
| [top](#flags) | :moldova: | `:moldova:` | :montenegro: | `:montenegro:` |
| [top](#flags) | :st_martin: | `:st_martin:` | :madagascar: | `:madagascar:` |
| [top](#flags) | :marshall_islands: | `:marshall_islands:` | :macedonia: | `:macedonia:` |
| [top](#flags) | :mali: | `:mali:` | :myanmar: | `:myanmar:` |
| [top](#flags) | :mongolia: | `:mongolia:` | :macau: | `:macau:` |
| [top](#flags) | :northern_mariana_islands: | `:northern_mariana_islands:` | :martinique: | `:martinique:` |
| [top](#flags) | :mauritania: | `:mauritania:` | :montserrat: | `:montserrat:` |
| [top](#flags) | :malta: | `:malta:` | :mauritius: | `:mauritius:` |
| [top](#flags) | :maldives: | `:maldives:` | :malawi: | `:malawi:` |
| [top](#flags) | :mexico: | `:mexico:` | :malaysia: | `:malaysia:` |
| [top](#flags) | :mozambique: | `:mozambique:` | :namibia: | `:namibia:` |
| [top](#flags) | :new_caledonia: | `:new_caledonia:` | :niger: | `:niger:` |
| [top](#flags) | :norfolk_island: | `:norfolk_island:` | :nigeria: | `:nigeria:` |
| [top](#flags) | :nicaragua: | `:nicaragua:` | :netherlands: | `:netherlands:` |
| [top](#flags) | :norway: | `:norway:` | :nepal: | `:nepal:` |
| [top](#flags) | :nauru: | `:nauru:` | :niue: | `:niue:` |
| [top](#flags) | :new_zealand: | `:new_zealand:` | :oman: | `:oman:` |
| [top](#flags) | :panama: | `:panama:` | :peru: | `:peru:` |
| [top](#flags) | :french_polynesia: | `:french_polynesia:` | :papua_new_guinea: | `:papua_new_guinea:` |
| [top](#flags) | :philippines: | `:philippines:` | :pakistan: | `:pakistan:` |
| [top](#flags) | :poland: | `:poland:` | :st_pierre_miquelon: | `:st_pierre_miquelon:` |
| [top](#flags) | :pitcairn_islands: | `:pitcairn_islands:` | :puerto_rico: | `:puerto_rico:` |
| [top](#flags) | :palestinian_territories: | `:palestinian_territories:` | :portugal: | `:portugal:` |
| [top](#flags) | :palau: | `:palau:` | :paraguay: | `:paraguay:` |
| [top](#flags) | :qatar: | `:qatar:` | :reunion: | `:reunion:` |
| [top](#flags) | :romania: | `:romania:` | :serbia: | `:serbia:` |
| [top](#flags) | :ru: | `:ru:` | :rwanda: | `:rwanda:` |
| [top](#flags) | :saudi_arabia: | `:saudi_arabia:` | :solomon_islands: | `:solomon_islands:` |
| [top](#flags) | :seychelles: | `:seychelles:` | :sudan: | `:sudan:` |
| [top](#flags) | :sweden: | `:sweden:` | :singapore: | `:singapore:` |
| [top](#flags) | :st_helena: | `:st_helena:` | :slovenia: | `:slovenia:` |
| [top](#flags) | :svalbard_jan_mayen: | `:svalbard_jan_mayen:` | :slovakia: | `:slovakia:` |
| [top](#flags) | :sierra_leone: | `:sierra_leone:` | :san_marino: | `:san_marino:` |
| [top](#flags) | :senegal: | `:senegal:` | :somalia: | `:somalia:` |
| [top](#flags) | :suriname: | `:suriname:` | :south_sudan: | `:south_sudan:` |
| [top](#flags) | :sao_tome_principe: | `:sao_tome_principe:` | :el_salvador: | `:el_salvador:` |
| [top](#flags) | :sint_maarten: | `:sint_maarten:` | :syria: | `:syria:` |
| [top](#flags) | :swaziland: | `:swaziland:` | :tristan_da_cunha: | `:tristan_da_cunha:` |
| [top](#flags) | :turks_caicos_islands: | `:turks_caicos_islands:` | :chad: | `:chad:` |
| [top](#flags) | :french_southern_territories: | `:french_southern_territories:` | :togo: | `:togo:` |
| [top](#flags) | :thailand: | `:thailand:` | :tajikistan: | `:tajikistan:` |
| [top](#flags) | :tokelau: | `:tokelau:` | :timor_leste: | `:timor_leste:` |
| [top](#flags) | :turkmenistan: | `:turkmenistan:` | :tunisia: | `:tunisia:` |
| [top](#flags) | :tonga: | `:tonga:` | :tr: | `:tr:` |
| [top](#flags) | :trinidad_tobago: | `:trinidad_tobago:` | :tuvalu: | `:tuvalu:` |
| [top](#flags) | :taiwan: | `:taiwan:` | :tanzania: | `:tanzania:` |
| [top](#flags) | :ukraine: | `:ukraine:` | :uganda: | `:uganda:` |
| [top](#flags) | :us_outlying_islands: | `:us_outlying_islands:` | :united_nations: | `:united_nations:` |
| [top](#flags) | :us: | `:us:` | :uruguay: | `:uruguay:` |
| [top](#flags) | :uzbekistan: | `:uzbekistan:` | :vatican_city: | `:vatican_city:` |
| [top](#flags) | :st_vincent_grenadines: | `:st_vincent_grenadines:` | :venezuela: | `:venezuela:` |
| [top](#flags) | :british_virgin_islands: | `:british_virgin_islands:` | :us_virgin_islands: | `:us_virgin_islands:` |
| [top](#flags) | :vietnam: | `:vietnam:` | :vanuatu: | `:vanuatu:` |
| [top](#flags) | :wallis_futuna: | `:wallis_futuna:` | :samoa: | `:samoa:` |
| [top](#flags) | :kosovo: | `:kosovo:` | :yemen: | `:yemen:` |
| [top](#flags) | :mayotte: | `:mayotte:` | :south_africa: | `:south_africa:` |
| [top](#flags) | :zambia: | `:zambia:` | :zimbabwe: | `:zimbabwe:` |

### Smileys & Emotion

#### Visage souriant

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#smileys--emotion) | :grinning: | `:grinning:` | :smiley: | `:smiley:` |
| [top](#smileys--emotion) | :smile: | `:smile:` | :grin: | `:grin:` |
| [top](#smileys--emotion) | :laughing: | `:laughing:` <br /> `:satisfied:` | :sweat_smile: | `:sweat_smile:` |
| [top](#smileys--emotion) | :rofl: | `:rofl:` | :joy: | `:joy:` |
| [top](#smileys--emotion) | :slightly_smiling_face: | `:slightly_smiling_face:` | :upside_down_face: | `:upside_down_face:` |
| [top](#smileys--emotion) | :wink: | `:wink:` | :blush: | `:blush:` |
| [top](#smileys--emotion) | :innocent: | `:innocent:` | | |

#### Visage affecté

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#smileys--emotion) | :smiling_face_with_three_hearts: | `:smiling_face_with_three_hearts:` | :heart_eyes: | `:heart_eyes:` |
| [top](#smileys--emotion) | :star_struck: | `:star_struck:` | :kissing_heart: | `:kissing_heart:` |
| [top](#smileys--emotion) | :kissing: | `:kissing:` | :relaxed: | `:relaxed:` |
| [top](#smileys--emotion) | :kissing_closed_eyes: | `:kissing_closed_eyes:` | :kissing_smiling_eyes: | `:kissing_smiling_eyes:` |
| [top](#smileys--emotion) | :smiling_face_with_tear: | `:smiling_face_with_tear:` | | |

#### Visage + langue

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#smileys--emotion) | :yum: | `:yum:` | :stuck_out_tongue: | `:stuck_out_tongue:` |
| [top](#smileys--emotion) | :stuck_out_tongue_winking_eye: | `:stuck_out_tongue_winking_eye:` | :zany_face: | `:zany_face:` |
| [top](#smileys--emotion) | :stuck_out_tongue_closed_eyes: | `:stuck_out_tongue_closed_eyes:` | :money_mouth_face: | `:money_mouth_face:` |

#### Main sur visage

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#smileys--emotion) | :hugs: | `:hugs:` | :hand_over_mouth: | `:hand_over_mouth:` |
| [top](#smileys--emotion) | :shushing_face: | `:shushing_face:` | :thinking: | `:thinking:` |

#### Visage neutre

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#smileys--emotion) | :zipper_mouth_face: | `:zipper_mouth_face:` | :raised_eyebrow: | `:raised_eyebrow:` |
| [top](#smileys--emotion) | :neutral_face: | `:neutral_face:` | :expressionless: | `:expressionless:` |
| [top](#smileys--emotion) | :no_mouth: | `:no_mouth:` | :face_in_clouds: | `:face_in_clouds:` |
| [top](#smileys--emotion) | :smirk: | `:smirk:` | :unamused: | `:unamused:` |
| [top](#smileys--emotion) | :roll_eyes: | `:roll_eyes:` | :grimacing: | `:grimacing:` |
| [top](#smileys--emotion) | :face_exhaling: | `:face_exhaling:` | :lying_face: | `:lying_face:` |

#### Visage endormi

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#smileys--emotion) | :relieved: | `:relieved:` | :pensive: | `:pensive:` |
| [top](#smileys--emotion) | :sleepy: | `:sleepy:` | :drooling_face: | `:drooling_face:` |
| [top](#smileys--emotion) | :sleeping: | `:sleeping:` | | |

#### Malaise du visage

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#smileys--emotion) | :mask: | `:mask:` | :face_with_thermometer: | `:face_with_thermometer:` |
| [top](#smileys--emotion) | :face_with_head_bandage: | `:face_with_head_bandage:` | :nauseated_face: | `:nauseated_face:` |
| [top](#smileys--emotion) | :vomiting_face: | `:vomiting_face:` | :sneezing_face: | `:sneezing_face:` |
| [top](#smileys--emotion) | :hot_face: | `:hot_face:` | :cold_face: | `:cold_face:` |
| [top](#smileys--emotion) | :woozy_face: | `:woozy_face:` | :dizzy_face: | `:dizzy_face:` |
| [top](#smileys--emotion) | :face_with_spiral_eyes: | `:face_with_spiral_eyes:` | :exploding_head: | `:exploding_head:` |

#### Visage + chapeau

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#smileys--emotion) | :cowboy_hat_face: | `:cowboy_hat_face:` | :partying_face: | `:partying_face:` |
| [top](#smileys--emotion) | :disguised_face: | `:disguised_face:` | | |

#### Visage + lunettes

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#smileys--emotion) | :sunglasses: | `:sunglasses:` | :nerd_face: | `:nerd_face:` |
| [top](#smileys--emotion) | :monocle_face: | `:monocle_face:` | | |

#### Visage concerné

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#smileys--emotion) | :confused: | `:confused:` | :worried: | `:worried:` |
| [top](#smileys--emotion) | :slightly_frowning_face: | `:slightly_frowning_face:` | :frowning_face: | `:frowning_face:` |
| [top](#smileys--emotion) | :open_mouth: | `:open_mouth:` | :hushed: | `:hushed:` |
| [top](#smileys--emotion) | :astonished: | `:astonished:` | :flushed: | `:flushed:` |
| [top](#smileys--emotion) | :pleading_face: | `:pleading_face:` | :frowning: | `:frowning:` |
| [top](#smileys--emotion) | :anguished: | `:anguished:` | :fearful: | `:fearful:` |
| [top](#smileys--emotion) | :cold_sweat: | `:cold_sweat:` | :disappointed_relieved: | `:disappointed_relieved:` |
| [top](#smileys--emotion) | :cry: | `:cry:` | :sob: | `:sob:` |
| [top](#smileys--emotion) | :scream: | `:scream:` | :confounded: | `:confounded:` |
| [top](#smileys--emotion) | :persevere: | `:persevere:` | :disappointed: | `:disappointed:` |
| [top](#smileys--emotion) | :sweat: | `:sweat:` | :weary: | `:weary:` |
| [top](#smileys--emotion) | :tired_face: | `:tired_face:` | :yawning_face: | `:yawning_face:` |

#### Visage négatif

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#smileys--emotion) | :triumph: | `:triumph:` | :pout: | `:pout:` <br /> `:rage:` |
| [top](#smileys--emotion) | :angry: | `:angry:` | :cursing_face: | `:cursing_face:` |
| [top](#smileys--emotion) | :smiling_imp: | `:smiling_imp:` | :imp: | `:imp:` |
| [top](#smileys--emotion) | :skull: | `:skull:` | :skull_and_crossbones: | `:skull_and_crossbones:` |

#### Visage deguisé

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#smileys--emotion) | :hankey: | `:hankey:` <br /> `:poop:` <br /> `:shit:` | :clown_face: | `:clown_face:` |
| [top](#smileys--emotion) | :japanese_ogre: | `:japanese_ogre:` | :japanese_goblin: | `:japanese_goblin:` |
| [top](#smileys--emotion) | :ghost: | `:ghost:` | :alien: | `:alien:` |
| [top](#smileys--emotion) | :space_invader: | `:space_invader:` | :robot: | `:robot:` |

#### Visage de chat

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#smileys--emotion) | :smiley_cat: | `:smiley_cat:` | :smile_cat: | `:smile_cat:` |
| [top](#smileys--emotion) | :joy_cat: | `:joy_cat:` | :heart_eyes_cat: | `:heart_eyes_cat:` |
| [top](#smileys--emotion) | :smirk_cat: | `:smirk_cat:` | :kissing_cat: | `:kissing_cat:` |
| [top](#smileys--emotion) | :scream_cat: | `:scream_cat:` | :crying_cat_face: | `:crying_cat_face:` |
| [top](#smileys--emotion) | :pouting_cat: | `:pouting_cat:` | | |

#### Visage de singe

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#smileys--emotion) | :see_no_evil: | `:see_no_evil:` | :hear_no_evil: | `:hear_no_evil:` |
| [top](#smileys--emotion) | :speak_no_evil: | `:speak_no_evil:` | | |

#### Heart

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#smileys--emotion) | :love_letter: | `:love_letter:` | :cupid: | `:cupid:` |
| [top](#smileys--emotion) | :gift_heart: | `:gift_heart:` | :sparkling_heart: | `:sparkling_heart:` |
| [top](#smileys--emotion) | :heartpulse: | `:heartpulse:` | :heartbeat: | `:heartbeat:` |
| [top](#smileys--emotion) | :revolving_hearts: | `:revolving_hearts:` | :two_hearts: | `:two_hearts:` |
| [top](#smileys--emotion) | :heart_decoration: | `:heart_decoration:` | :heavy_heart_exclamation: | `:heavy_heart_exclamation:` |
| [top](#smileys--emotion) | :broken_heart: | `:broken_heart:` | :heart_on_fire: | `:heart_on_fire:` |
| [top](#smileys--emotion) | :mending_heart: | `:mending_heart:` | :heart: | `:heart:` |
| [top](#smileys--emotion) | :orange_heart: | `:orange_heart:` | :yellow_heart: | `:yellow_heart:` |
| [top](#smileys--emotion) | :green_heart: | `:green_heart:` | :blue_heart: | `:blue_heart:` |
| [top](#smileys--emotion) | :purple_heart: | `:purple_heart:` | :brown_heart: | `:brown_heart:` |
| [top](#smileys--emotion) | :black_heart: | `:black_heart:` | :white_heart: | `:white_heart:` |

#### Emotion

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#smileys--emotion) | :kiss: | `:kiss:` | :100: | `:100:` |
| [top](#smileys--emotion) | :anger: | `:anger:` | :boom: | `:boom:` <br /> `:collision:` |
| [top](#smileys--emotion) | :dizzy: | `:dizzy:` | :sweat_drops: | `:sweat_drops:` |
| [top](#smileys--emotion) | :dash: | `:dash:` | :hole: | `:hole:` |
| [top](#smileys--emotion) | :speech_balloon: | `:speech_balloon:` | :eye_speech_bubble: | `:eye_speech_bubble:` |
| [top](#smileys--emotion) | :left_speech_bubble: | `:left_speech_bubble:` | :right_anger_bubble: | `:right_anger_bubble:` |
| [top](#smileys--emotion) | :thought_balloon: | `:thought_balloon:` | :zzz: | `:zzz:` |

### Personnes et Corps

#### Doigts de main Ouverts

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#people--body) | :wave: | `:wave:` | :raised_back_of_hand: | `:raised_back_of_hand:` |
| [top](#people--body) | :raised_hand_with_fingers_splayed: | `:raised_hand_with_fingers_splayed:` | :hand: | `:hand:` <br /> `:raised_hand:` |
| [top](#people--body) | :vulcan_salute: | `:vulcan_salute:` | | |

#### Doigts de main partiels

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#people--body) | :ok_hand: | `:ok_hand:` | :pinched_fingers: | `:pinched_fingers:` |
| [top](#people--body) | :pinching_hand: | `:pinching_hand:` | :v: | `:v:` |
| [top](#people--body) | :crossed_fingers: | `:crossed_fingers:` | :love_you_gesture: | `:love_you_gesture:` |
| [top](#people--body) | :metal: | `:metal:` | :call_me_hand: | `:call_me_hand:` |

#### Doigt simple main

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#people--body) | :point_left: | `:point_left:` | :point_right: | `:point_right:` |
| [top](#people--body) | :point_up_2: | `:point_up_2:` | :fu: | `:fu:` <br /> `:middle_finger:` |
| [top](#people--body) | :point_down: | `:point_down:` | :point_up: | `:point_up:` |

#### Doigts main fermés

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#people--body) | :+1: | `:+1:` <br /> `:thumbsup:` | :-1: | `:-1:` <br /> `:thumbsdown:` |
| [top](#people--body) | :fist: | `:fist:` <br /> `:fist_raised:` | :facepunch: | `:facepunch:` <br /> `:fist_oncoming:` <br /> `:punch:` |
| [top](#people--body) | :fist_left: | `:fist_left:` | :fist_right: | `:fist_right:` |

#### Mains

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#people--body) | :clap: | `:clap:` | :raised_hands: | `:raised_hands:` |
| [top](#people--body) | :open_hands: | `:open_hands:` | :palms_up_together: | `:palms_up_together:` |
| [top](#people--body) | :handshake: | `:handshake:` | :pray: | `:pray:` |

#### Main + accessoire

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#people--body) | :writing_hand: | `:writing_hand:` | :nail_care: | `:nail_care:` |
| [top](#people--body) | :selfie: | `:selfie:` | | |

#### Parties du corps

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#people--body) | :muscle: | `:muscle:` | :mechanical_arm: | `:mechanical_arm:` |
| [top](#people--body) | :mechanical_leg: | `:mechanical_leg:` | :leg: | `:leg:` |
| [top](#people--body) | :foot: | `:foot:` | :ear: | `:ear:` |
| [top](#people--body) | :ear_with_hearing_aid: | `:ear_with_hearing_aid:` | :nose: | `:nose:` |
| [top](#people--body) | :brain: | `:brain:` | :anatomical_heart: | `:anatomical_heart:` |
| [top](#people--body) | :lungs: | `:lungs:` | :tooth: | `:tooth:` |
| [top](#people--body) | :bone: | `:bone:` | :eyes: | `:eyes:` |
| [top](#people--body) | :eye: | `:eye:` | :tongue: | `:tongue:` |
| [top](#people--body) | :lips: | `:lips:` | | |

#### Personne

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#people--body) | :baby: | `:baby:` | :child: | `:child:` |
| [top](#people--body) | :boy: | `:boy:` | :girl: | `:girl:` |
| [top](#people--body) | :adult: | `:adult:` | :blond_haired_person: | `:blond_haired_person:` |
| [top](#people--body) | :man: | `:man:` | :bearded_person: | `:bearded_person:` |
| [top](#people--body) | :man_beard: | `:man_beard:` | :woman_beard: | `:woman_beard:` |
| [top](#people--body) | :red_haired_man: | `:red_haired_man:` | :curly_haired_man: | `:curly_haired_man:` |
| [top](#people--body) | :white_haired_man: | `:white_haired_man:` | :bald_man: | `:bald_man:` |
| [top](#people--body) | :woman: | `:woman:` | :red_haired_woman: | `:red_haired_woman:` |
| [top](#people--body) | :person_red_hair: | `:person_red_hair:` | :curly_haired_woman: | `:curly_haired_woman:` |
| [top](#people--body) | :person_curly_hair: | `:person_curly_hair:` | :white_haired_woman: | `:white_haired_woman:` |
| [top](#people--body) | :person_white_hair: | `:person_white_hair:` | :bald_woman: | `:bald_woman:` |
| [top](#people--body) | :person_bald: | `:person_bald:` | :blond_haired_woman: | `:blond_haired_woman:` <br /> `:blonde_woman:` |
| [top](#people--body) | :blond_haired_man: | `:blond_haired_man:` | :older_adult: | `:older_adult:` |
| [top](#people--body) | :older_man: | `:older_man:` | :older_woman: | `:older_woman:` |

#### Personne +  Geste

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#people--body) | :frowning_person: | `:frowning_person:` | :frowning_man: | `:frowning_man:` |
| [top](#people--body) | :frowning_woman: | `:frowning_woman:` | :pouting_face: | `:pouting_face:` |
| [top](#people--body) | :pouting_man: | `:pouting_man:` | :pouting_woman: | `:pouting_woman:` |
| [top](#people--body) | :no_good: | `:no_good:` | :ng_man: | `:ng_man:` <br /> `:no_good_man:` |
| [top](#people--body) | :ng_woman: | `:ng_woman:` <br /> `:no_good_woman:` | :ok_person: | `:ok_person:` |
| [top](#people--body) | :ok_man: | `:ok_man:` | :ok_woman: | `:ok_woman:` |
| [top](#people--body) | :information_desk_person: | `:information_desk_person:` <br /> `:tipping_hand_person:` | :sassy_man: | `:sassy_man:` <br /> `:tipping_hand_man:` |
| [top](#people--body) | :sassy_woman: | `:sassy_woman:` <br /> `:tipping_hand_woman:` | :raising_hand: | `:raising_hand:` |
| [top](#people--body) | :raising_hand_man: | `:raising_hand_man:` | :raising_hand_woman: | `:raising_hand_woman:` |
| [top](#people--body) | :deaf_person: | `:deaf_person:` | :deaf_man: | `:deaf_man:` |
| [top](#people--body) | :deaf_woman: | `:deaf_woman:` | :bow: | `:bow:` |
| [top](#people--body) | :bowing_man: | `:bowing_man:` | :bowing_woman: | `:bowing_woman:` |
| [top](#people--body) | :facepalm: | `:facepalm:` | :man_facepalming: | `:man_facepalming:` |
| [top](#people--body) | :woman_facepalming: | `:woman_facepalming:` | :shrug: | `:shrug:` |
| [top](#people--body) | :man_shrugging: | `:man_shrugging:` | :woman_shrugging: | `:woman_shrugging:` |

#### Rôle des personnes

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#people--body) | :health_worker: | `:health_worker:` | :man_health_worker: | `:man_health_worker:` |
| [top](#people--body) | :woman_health_worker: | `:woman_health_worker:` | :student: | `:student:` |
| [top](#people--body) | :man_student: | `:man_student:` | :woman_student: | `:woman_student:` |
| [top](#people--body) | :teacher: | `:teacher:` | :man_teacher: | `:man_teacher:` |
| [top](#people--body) | :woman_teacher: | `:woman_teacher:` | :judge: | `:judge:` |
| [top](#people--body) | :man_judge: | `:man_judge:` | :woman_judge: | `:woman_judge:` |
| [top](#people--body) | :farmer: | `:farmer:` | :man_farmer: | `:man_farmer:` |
| [top](#people--body) | :woman_farmer: | `:woman_farmer:` | :cook: | `:cook:` |
| [top](#people--body) | :man_cook: | `:man_cook:` | :woman_cook: | `:woman_cook:` |
| [top](#people--body) | :mechanic: | `:mechanic:` | :man_mechanic: | `:man_mechanic:` |
| [top](#people--body) | :woman_mechanic: | `:woman_mechanic:` | :factory_worker: | `:factory_worker:` |
| [top](#people--body) | :man_factory_worker: | `:man_factory_worker:` | :woman_factory_worker: | `:woman_factory_worker:` |
| [top](#people--body) | :office_worker: | `:office_worker:` | :man_office_worker: | `:man_office_worker:` |
| [top](#people--body) | :woman_office_worker: | `:woman_office_worker:` | :scientist: | `:scientist:` |
| [top](#people--body) | :man_scientist: | `:man_scientist:` | :woman_scientist: | `:woman_scientist:` |
| [top](#people--body) | :technologist: | `:technologist:` | :man_technologist: | `:man_technologist:` |
| [top](#people--body) | :woman_technologist: | `:woman_technologist:` | :singer: | `:singer:` |
| [top](#people--body) | :man_singer: | `:man_singer:` | :woman_singer: | `:woman_singer:` |
| [top](#people--body) | :artist: | `:artist:` | :man_artist: | `:man_artist:` |
| [top](#people--body) | :woman_artist: | `:woman_artist:` | :pilot: | `:pilot:` |
| [top](#people--body) | :man_pilot: | `:man_pilot:` | :woman_pilot: | `:woman_pilot:` |
| [top](#people--body) | :astronaut: | `:astronaut:` | :man_astronaut: | `:man_astronaut:` |
| [top](#people--body) | :woman_astronaut: | `:woman_astronaut:` | :firefighter: | `:firefighter:` |
| [top](#people--body) | :man_firefighter: | `:man_firefighter:` | :woman_firefighter: | `:woman_firefighter:` |
| [top](#people--body) | :cop: | `:cop:` <br /> `:police_officer:` | :policeman: | `:policeman:` |
| [top](#people--body) | :policewoman: | `:policewoman:` | :detective: | `:detective:` |
| [top](#people--body) | :male_detective: | `:male_detective:` | :female_detective: | `:female_detective:` |
| [top](#people--body) | :guard: | `:guard:` | :guardsman: | `:guardsman:` |
| [top](#people--body) | :guardswoman: | `:guardswoman:` | :ninja: | `:ninja:` |
| [top](#people--body) | :construction_worker: | `:construction_worker:` | :construction_worker_man: | `:construction_worker_man:` |
| [top](#people--body) | :construction_worker_woman: | `:construction_worker_woman:` | :prince: | `:prince:` |
| [top](#people--body) | :princess: | `:princess:` | :person_with_turban: | `:person_with_turban:` |
| [top](#people--body) | :man_with_turban: | `:man_with_turban:` | :woman_with_turban: | `:woman_with_turban:` |
| [top](#people--body) | :man_with_gua_pi_mao: | `:man_with_gua_pi_mao:` | :woman_with_headscarf: | `:woman_with_headscarf:` |
| [top](#people--body) | :person_in_tuxedo: | `:person_in_tuxedo:` | :man_in_tuxedo: | `:man_in_tuxedo:` |
| [top](#people--body) | :woman_in_tuxedo: | `:woman_in_tuxedo:` | :person_with_veil: | `:person_with_veil:` |
| [top](#people--body) | :man_with_veil: | `:man_with_veil:` | :bride_with_veil: | `:bride_with_veil:` <br /> `:woman_with_veil:` |
| [top](#people--body) | :pregnant_woman: | `:pregnant_woman:` | :breast_feeding: | `:breast_feeding:` |
| [top](#people--body) | :woman_feeding_baby: | `:woman_feeding_baby:` | :man_feeding_baby: | `:man_feeding_baby:` |
| [top](#people--body) | :person_feeding_baby: | `:person_feeding_baby:` | | |

#### Personne Fantaisie

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#people--body) | :angel: | `:angel:` | :santa: | `:santa:` |
| [top](#people--body) | :mrs_claus: | `:mrs_claus:` | :mx_claus: | `:mx_claus:` |
| [top](#people--body) | :superhero: | `:superhero:` | :superhero_man: | `:superhero_man:` |
| [top](#people--body) | :superhero_woman: | `:superhero_woman:` | :supervillain: | `:supervillain:` |
| [top](#people--body) | :supervillain_man: | `:supervillain_man:` | :supervillain_woman: | `:supervillain_woman:` |
| [top](#people--body) | :mage: | `:mage:` | :mage_man: | `:mage_man:` |
| [top](#people--body) | :mage_woman: | `:mage_woman:` | :fairy: | `:fairy:` |
| [top](#people--body) | :fairy_man: | `:fairy_man:` | :fairy_woman: | `:fairy_woman:` |
| [top](#people--body) | :vampire: | `:vampire:` | :vampire_man: | `:vampire_man:` |
| [top](#people--body) | :vampire_woman: | `:vampire_woman:` | :merperson: | `:merperson:` |
| [top](#people--body) | :merman: | `:merman:` | :mermaid: | `:mermaid:` |
| [top](#people--body) | :elf: | `:elf:` | :elf_man: | `:elf_man:` |
| [top](#people--body) | :elf_woman: | `:elf_woman:` | :genie: | `:genie:` |
| [top](#people--body) | :genie_man: | `:genie_man:` | :genie_woman: | `:genie_woman:` |
| [top](#people--body) | :zombie: | `:zombie:` | :zombie_man: | `:zombie_man:` |
| [top](#people--body) | :zombie_woman: | `:zombie_woman:` | | |

#### Personne activité

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#people--body) | :massage: | `:massage:` | :massage_man: | `:massage_man:` |
| [top](#people--body) | :massage_woman: | `:massage_woman:` | :haircut: | `:haircut:` |
| [top](#people--body) | :haircut_man: | `:haircut_man:` | :haircut_woman: | `:haircut_woman:` |
| [top](#people--body) | :walking: | `:walking:` | :walking_man: | `:walking_man:` |
| [top](#people--body) | :walking_woman: | `:walking_woman:` | :standing_person: | `:standing_person:` |
| [top](#people--body) | :standing_man: | `:standing_man:` | :standing_woman: | `:standing_woman:` |
| [top](#people--body) | :kneeling_person: | `:kneeling_person:` | :kneeling_man: | `:kneeling_man:` |
| [top](#people--body) | :kneeling_woman: | `:kneeling_woman:` | :person_with_probing_cane: | `:person_with_probing_cane:` |
| [top](#people--body) | :man_with_probing_cane: | `:man_with_probing_cane:` | :woman_with_probing_cane: | `:woman_with_probing_cane:` |
| [top](#people--body) | :person_in_motorized_wheelchair: | `:person_in_motorized_wheelchair:` | :man_in_motorized_wheelchair: | `:man_in_motorized_wheelchair:` |
| [top](#people--body) | :woman_in_motorized_wheelchair: | `:woman_in_motorized_wheelchair:` | :person_in_manual_wheelchair: | `:person_in_manual_wheelchair:` |
| [top](#people--body) | :man_in_manual_wheelchair: | `:man_in_manual_wheelchair:` | :woman_in_manual_wheelchair: | `:woman_in_manual_wheelchair:` |
| [top](#people--body) | :runner: | `:runner:` <br /> `:running:` | :running_man: | `:running_man:` |
| [top](#people--body) | :running_woman: | `:running_woman:` | :dancer: | `:dancer:` <br /> `:woman_dancing:` |
| [top](#people--body) | :man_dancing: | `:man_dancing:` | :business_suit_levitating: | `:business_suit_levitating:` |
| [top](#people--body) | :dancers: | `:dancers:` | :dancing_men: | `:dancing_men:` |
| [top](#people--body) | :dancing_women: | `:dancing_women:` | :sauna_person: | `:sauna_person:` |
| [top](#people--body) | :sauna_man: | `:sauna_man:` | :sauna_woman: | `:sauna_woman:` |
| [top](#people--body) | :climbing: | `:climbing:` | :climbing_man: | `:climbing_man:` |
| [top](#people--body) | :climbing_woman: | `:climbing_woman:` | | |

#### Personne Sport

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#people--body) | :person_fencing: | `:person_fencing:` | :horse_racing: | `:horse_racing:` |
| [top](#people--body) | :skier: | `:skier:` | :snowboarder: | `:snowboarder:` |
| [top](#people--body) | :golfing: | `:golfing:` | :golfing_man: | `:golfing_man:` |
| [top](#people--body) | :golfing_woman: | `:golfing_woman:` | :surfer: | `:surfer:` |
| [top](#people--body) | :surfing_man: | `:surfing_man:` | :surfing_woman: | `:surfing_woman:` |
| [top](#people--body) | :rowboat: | `:rowboat:` | :rowing_man: | `:rowing_man:` |
| [top](#people--body) | :rowing_woman: | `:rowing_woman:` | :swimmer: | `:swimmer:` |
| [top](#people--body) | :swimming_man: | `:swimming_man:` | :swimming_woman: | `:swimming_woman:` |
| [top](#people--body) | :bouncing_ball_person: | `:bouncing_ball_person:` | :basketball_man: | `:basketball_man:` <br /> `:bouncing_ball_man:` |
| [top](#people--body) | :basketball_woman: | `:basketball_woman:` <br /> `:bouncing_ball_woman:` | :weight_lifting: | `:weight_lifting:` |
| [top](#people--body) | :weight_lifting_man: | `:weight_lifting_man:` | :weight_lifting_woman: | `:weight_lifting_woman:` |
| [top](#people--body) | :bicyclist: | `:bicyclist:` | :biking_man: | `:biking_man:` |
| [top](#people--body) | :biking_woman: | `:biking_woman:` | :mountain_bicyclist: | `:mountain_bicyclist:` |
| [top](#people--body) | :mountain_biking_man: | `:mountain_biking_man:` | :mountain_biking_woman: | `:mountain_biking_woman:` |
| [top](#people--body) | :cartwheeling: | `:cartwheeling:` | :man_cartwheeling: | `:man_cartwheeling:` |
| [top](#people--body) | :woman_cartwheeling: | `:woman_cartwheeling:` | :wrestling: | `:wrestling:` |
| [top](#people--body) | :men_wrestling: | `:men_wrestling:` | :women_wrestling: | `:women_wrestling:` |
| [top](#people--body) | :water_polo: | `:water_polo:` | :man_playing_water_polo: | `:man_playing_water_polo:` |
| [top](#people--body) | :woman_playing_water_polo: | `:woman_playing_water_polo:` | :handball_person: | `:handball_person:` |
| [top](#people--body) | :man_playing_handball: | `:man_playing_handball:` | :woman_playing_handball: | `:woman_playing_handball:` |
| [top](#people--body) | :juggling_person: | `:juggling_person:` | :man_juggling: | `:man_juggling:` |
| [top](#people--body) | :woman_juggling: | `:woman_juggling:` | | |

#### Personne au repos

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#people--body) | :lotus_position: | `:lotus_position:` | :lotus_position_man: | `:lotus_position_man:` |
| [top](#people--body) | :lotus_position_woman: | `:lotus_position_woman:` | :bath: | `:bath:` |
| [top](#people--body) | :sleeping_bed: | `:sleeping_bed:` | | |

#### Famille

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#people--body) | :people_holding_hands: | `:people_holding_hands:` | :two_women_holding_hands: | `:two_women_holding_hands:` |
| [top](#people--body) | :couple: | `:couple:` | :two_men_holding_hands: | `:two_men_holding_hands:` |
| [top](#people--body) | :couplekiss: | `:couplekiss:` | :couplekiss_man_woman: | `:couplekiss_man_woman:` |
| [top](#people--body) | :couplekiss_man_man: | `:couplekiss_man_man:` | :couplekiss_woman_woman: | `:couplekiss_woman_woman:` |
| [top](#people--body) | :couple_with_heart: | `:couple_with_heart:` | :couple_with_heart_woman_man: | `:couple_with_heart_woman_man:` |
| [top](#people--body) | :couple_with_heart_man_man: | `:couple_with_heart_man_man:` | :couple_with_heart_woman_woman: | `:couple_with_heart_woman_woman:` |
| [top](#people--body) | :family_man_woman_boy: | `:family_man_woman_boy:` | :family_man_woman_girl: | `:family_man_woman_girl:` |
| [top](#people--body) | :family_man_woman_girl_boy: | `:family_man_woman_girl_boy:` | :family_man_woman_boy_boy: | `:family_man_woman_boy_boy:` |
| [top](#people--body) | :family_man_woman_girl_girl: | `:family_man_woman_girl_girl:` | :family_man_man_boy: | `:family_man_man_boy:` |
| [top](#people--body) | :family_man_man_girl: | `:family_man_man_girl:` | :family_man_man_girl_boy: | `:family_man_man_girl_boy:` |
| [top](#people--body) | :family_man_man_boy_boy: | `:family_man_man_boy_boy:` | :family_man_man_girl_girl: | `:family_man_man_girl_girl:` |
| [top](#people--body) | :family_woman_woman_boy: | `:family_woman_woman_boy:` | :family_woman_woman_girl: | `:family_woman_woman_girl:` |
| [top](#people--body) | :family_woman_woman_girl_boy: | `:family_woman_woman_girl_boy:` | :family_woman_woman_boy_boy: | `:family_woman_woman_boy_boy:` |
| [top](#people--body) | :family_woman_woman_girl_girl: | `:family_woman_woman_girl_girl:` | :family_man_boy: | `:family_man_boy:` |
| [top](#people--body) | :family_man_boy_boy: | `:family_man_boy_boy:` | :family_man_girl: | `:family_man_girl:` |
| [top](#people--body) | :family_man_girl_boy: | `:family_man_girl_boy:` | :family_man_girl_girl: | `:family_man_girl_girl:` |
| [top](#people--body) | :family_woman_boy: | `:family_woman_boy:` | :family_woman_boy_boy: | `:family_woman_boy_boy:` |
| [top](#people--body) | :family_woman_girl: | `:family_woman_girl:` | :family_woman_girl_boy: | `:family_woman_girl_boy:` |
| [top](#people--body) | :family_woman_girl_girl: | `:family_woman_girl_girl:` | | |

#### Symbole de personne

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#people--body) | :speaking_head: | `:speaking_head:` | :bust_in_silhouette: | `:bust_in_silhouette:` |
| [top](#people--body) | :busts_in_silhouette: | `:busts_in_silhouette:` | :people_hugging: | `:people_hugging:` |
| [top](#people--body) | :family: | `:family:` | :footprints: | `:footprints:` |

### Animals & Nature

#### Mammifère

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#animals--nature) | :monkey_face: | `:monkey_face:` | :monkey: | `:monkey:` |
| [top](#animals--nature) | :gorilla: | `:gorilla:` | :orangutan: | `:orangutan:` |
| [top](#animals--nature) | :dog: | `:dog:` | :dog2: | `:dog2:` |
| [top](#animals--nature) | :guide_dog: | `:guide_dog:` | :service_dog: | `:service_dog:` |
| [top](#animals--nature) | :poodle: | `:poodle:` | :wolf: | `:wolf:` |
| [top](#animals--nature) | :fox_face: | `:fox_face:` | :raccoon: | `:raccoon:` |
| [top](#animals--nature) | :cat: | `:cat:` | :cat2: | `:cat2:` |
| [top](#animals--nature) | :black_cat: | `:black_cat:` | :lion: | `:lion:` |
| [top](#animals--nature) | :tiger: | `:tiger:` | :tiger2: | `:tiger2:` |
| [top](#animals--nature) | :leopard: | `:leopard:` | :horse: | `:horse:` |
| [top](#animals--nature) | :racehorse: | `:racehorse:` | :unicorn: | `:unicorn:` |
| [top](#animals--nature) | :zebra: | `:zebra:` | :deer: | `:deer:` |
| [top](#animals--nature) | :bison: | `:bison:` | :cow: | `:cow:` |
| [top](#animals--nature) | :ox: | `:ox:` | :water_buffalo: | `:water_buffalo:` |
| [top](#animals--nature) | :cow2: | `:cow2:` | :pig: | `:pig:` |
| [top](#animals--nature) | :pig2: | `:pig2:` | :boar: | `:boar:` |
| [top](#animals--nature) | :pig_nose: | `:pig_nose:` | :ram: | `:ram:` |
| [top](#animals--nature) | :sheep: | `:sheep:` | :goat: | `:goat:` |
| [top](#animals--nature) | :dromedary_camel: | `:dromedary_camel:` | :camel: | `:camel:` |
| [top](#animals--nature) | :llama: | `:llama:` | :giraffe: | `:giraffe:` |
| [top](#animals--nature) | :elephant: | `:elephant:` | :mammoth: | `:mammoth:` |
| [top](#animals--nature) | :rhinoceros: | `:rhinoceros:` | :hippopotamus: | `:hippopotamus:` |
| [top](#animals--nature) | :mouse: | `:mouse:` | :mouse2: | `:mouse2:` |
| [top](#animals--nature) | :rat: | `:rat:` | :hamster: | `:hamster:` |
| [top](#animals--nature) | :rabbit: | `:rabbit:` | :rabbit2: | `:rabbit2:` |
| [top](#animals--nature) | :chipmunk: | `:chipmunk:` | :beaver: | `:beaver:` |
| [top](#animals--nature) | :hedgehog: | `:hedgehog:` | :bat: | `:bat:` |
| [top](#animals--nature) | :bear: | `:bear:` | :polar_bear: | `:polar_bear:` |
| [top](#animals--nature) | :koala: | `:koala:` | :panda_face: | `:panda_face:` |
| [top](#animals--nature) | :sloth: | `:sloth:` | :otter: | `:otter:` |
| [top](#animals--nature) | :skunk: | `:skunk:` | :kangaroo: | `:kangaroo:` |
| [top](#animals--nature) | :badger: | `:badger:` | :feet: | `:feet:` <br /> `:paw_prints:` |

#### Oiseau

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#animals--nature) | :turkey: | `:turkey:` | :chicken: | `:chicken:` |
| [top](#animals--nature) | :rooster: | `:rooster:` | :hatching_chick: | `:hatching_chick:` |
| [top](#animals--nature) | :baby_chick: | `:baby_chick:` | :hatched_chick: | `:hatched_chick:` |
| [top](#animals--nature) | :bird: | `:bird:` | :penguin: | `:penguin:` |
| [top](#animals--nature) | :dove: | `:dove:` | :eagle: | `:eagle:` |
| [top](#animals--nature) | :duck: | `:duck:` | :swan: | `:swan:` |
| [top](#animals--nature) | :owl: | `:owl:` | :dodo: | `:dodo:` |
| [top](#animals--nature) | :feather: | `:feather:` | :flamingo: | `:flamingo:` |
| [top](#animals--nature) | :peacock: | `:peacock:` | :parrot: | `:parrot:` |

#### Animal Amphibien

| ico | shortcode | |
| - | :-: | - | - |
| [top](#animals--nature) | :frog: | `:frog:` |

#### Animal Reptile

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#animals--nature) | :crocodile: | `:crocodile:` | :turtle: | `:turtle:` |
| [top](#animals--nature) | :lizard: | `:lizard:` | :snake: | `:snake:` |
| [top](#animals--nature) | :dragon_face: | `:dragon_face:` | :dragon: | `:dragon:` |
| [top](#animals--nature) | :sauropod: | `:sauropod:` | :t-rex: | `:t-rex:` |

#### Animal Marin

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#animals--nature) | :whale: | `:whale:` | :whale2: | `:whale2:` |
| [top](#animals--nature) | :dolphin: | `:dolphin:` <br /> `:flipper:` | :seal: | `:seal:` |
| [top](#animals--nature) | :fish: | `:fish:` | :tropical_fish: | `:tropical_fish:` |
| [top](#animals--nature) | :blowfish: | `:blowfish:` | :shark: | `:shark:` |
| [top](#animals--nature) | :octopus: | `:octopus:` | :shell: | `:shell:` |

#### Animal Insecte

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#animals--nature) | :snail: | `:snail:` | :butterfly: | `:butterfly:` |
| [top](#animals--nature) | :bug: | `:bug:` | :ant: | `:ant:` |
| [top](#animals--nature) | :bee: | `:bee:` <br /> `:honeybee:` | :beetle: | `:beetle:` |
| [top](#animals--nature) | :lady_beetle: | `:lady_beetle:` | :cricket: | `:cricket:` |
| [top](#animals--nature) | :cockroach: | `:cockroach:` | :spider: | `:spider:` |
| [top](#animals--nature) | :spider_web: | `:spider_web:` | :scorpion: | `:scorpion:` |
| [top](#animals--nature) | :mosquito: | `:mosquito:` | :fly: | `:fly:` |
| [top](#animals--nature) | :worm: | `:worm:` | :microbe: | `:microbe:` |

#### Fleurs végétales

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#animals--nature) | :bouquet: | `:bouquet:` | :cherry_blossom: | `:cherry_blossom:` |
| [top](#animals--nature) | :white_flower: | `:white_flower:` | :rosette: | `:rosette:` |
| [top](#animals--nature) | :rose: | `:rose:` | :wilted_flower: | `:wilted_flower:` |
| [top](#animals--nature) | :hibiscus: | `:hibiscus:` | :sunflower: | `:sunflower:` |
| [top](#animals--nature) | :blossom: | `:blossom:` | :tulip: | `:tulip:` |

#### Autres plantes

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#animals--nature) | :seedling: | `:seedling:` | :potted_plant: | `:potted_plant:` |
| [top](#animals--nature) | :evergreen_tree: | `:evergreen_tree:` | :deciduous_tree: | `:deciduous_tree:` |
| [top](#animals--nature) | :palm_tree: | `:palm_tree:` | :cactus: | `:cactus:` |
| [top](#animals--nature) | :ear_of_rice: | `:ear_of_rice:` | :herb: | `:herb:` |
| [top](#animals--nature) | :shamrock: | `:shamrock:` | :four_leaf_clover: | `:four_leaf_clover:` |
| [top](#animals--nature) | :maple_leaf: | `:maple_leaf:` | :fallen_leaf: | `:fallen_leaf:` |
| [top](#animals--nature) | :leaves: | `:leaves:` | :mushroom: | `:mushroom:` |

### Alimentation et boissons

#### Fruits

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#food--drink) | :grapes: | `:grapes:` | :melon: | `:melon:` |
| [top](#food--drink) | :watermelon: | `:watermelon:` | :mandarin: | `:mandarin:` <br /> `:orange:` <br /> `:tangerine:` |
| [top](#food--drink) | :lemon: | `:lemon:` | :banana: | `:banana:` |
| [top](#food--drink) | :pineapple: | `:pineapple:` | :mango: | `:mango:` |
| [top](#food--drink) | :apple: | `:apple:` | :green_apple: | `:green_apple:` |
| [top](#food--drink) | :pear: | `:pear:` | :peach: | `:peach:` |
| [top](#food--drink) | :cherries: | `:cherries:` | :strawberry: | `:strawberry:` |
| [top](#food--drink) | :blueberries: | `:blueberries:` | :kiwi_fruit: | `:kiwi_fruit:` |
| [top](#food--drink) | :tomato: | `:tomato:` | :olive: | `:olive:` |
| [top](#food--drink) | :coconut: | `:coconut:` | | |

#### Légumes

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#food--drink) | :avocado: | `:avocado:` | :eggplant: | `:eggplant:` |
| [top](#food--drink) | :potato: | `:potato:` | :carrot: | `:carrot:` |
| [top](#food--drink) | :corn: | `:corn:` | :hot_pepper: | `:hot_pepper:` |
| [top](#food--drink) | :bell_pepper: | `:bell_pepper:` | :cucumber: | `:cucumber:` |
| [top](#food--drink) | :leafy_green: | `:leafy_green:` | :broccoli: | `:broccoli:` |
| [top](#food--drink) | :garlic: | `:garlic:` | :onion: | `:onion:` |
| [top](#food--drink) | :peanuts: | `:peanuts:` | :chestnut: | `:chestnut:` |

#### Aliments préparés

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#food--drink) | :bread: | `:bread:` | :croissant: | `:croissant:` |
| [top](#food--drink) | :baguette_bread: | `:baguette_bread:` | :flatbread: | `:flatbread:` |
| [top](#food--drink) | :pretzel: | `:pretzel:` | :bagel: | `:bagel:` |
| [top](#food--drink) | :pancakes: | `:pancakes:` | :waffle: | `:waffle:` |
| [top](#food--drink) | :cheese: | `:cheese:` | :meat_on_bone: | `:meat_on_bone:` |
| [top](#food--drink) | :poultry_leg: | `:poultry_leg:` | :cut_of_meat: | `:cut_of_meat:` |
| [top](#food--drink) | :bacon: | `:bacon:` | :hamburger: | `:hamburger:` |
| [top](#food--drink) | :fries: | `:fries:` | :pizza: | `:pizza:` |
| [top](#food--drink) | :hotdog: | `:hotdog:` | :sandwich: | `:sandwich:` |
| [top](#food--drink) | :taco: | `:taco:` | :burrito: | `:burrito:` |
| [top](#food--drink) | :tamale: | `:tamale:` | :stuffed_flatbread: | `:stuffed_flatbread:` |
| [top](#food--drink) | :falafel: | `:falafel:` | :egg: | `:egg:` |
| [top](#food--drink) | :fried_egg: | `:fried_egg:` | :shallow_pan_of_food: | `:shallow_pan_of_food:` |
| [top](#food--drink) | :stew: | `:stew:` | :fondue: | `:fondue:` |
| [top](#food--drink) | :bowl_with_spoon: | `:bowl_with_spoon:` | :green_salad: | `:green_salad:` |
| [top](#food--drink) | :popcorn: | `:popcorn:` | :butter: | `:butter:` |
| [top](#food--drink) | :salt: | `:salt:` | :canned_food: | `:canned_food:` |

#### Alimentation asiatique

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#food--drink) | :bento: | `:bento:` | :rice_cracker: | `:rice_cracker:` |
| [top](#food--drink) | :rice_ball: | `:rice_ball:` | :rice: | `:rice:` |
| [top](#food--drink) | :curry: | `:curry:` | :ramen: | `:ramen:` |
| [top](#food--drink) | :spaghetti: | `:spaghetti:` | :sweet_potato: | `:sweet_potato:` |
| [top](#food--drink) | :oden: | `:oden:` | :sushi: | `:sushi:` |
| [top](#food--drink) | :fried_shrimp: | `:fried_shrimp:` | :fish_cake: | `:fish_cake:` |
| [top](#food--drink) | :moon_cake: | `:moon_cake:` | :dango: | `:dango:` |
| [top](#food--drink) | :dumpling: | `:dumpling:` | :fortune_cookie: | `:fortune_cookie:` |
| [top](#food--drink) | :takeout_box: | `:takeout_box:` | | |

#### Aliment de la mer

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#food--drink) | :crab: | `:crab:` | :lobster: | `:lobster:` |
| [top](#food--drink) | :shrimp: | `:shrimp:` | :squid: | `:squid:` |
| [top](#food--drink) | :oyster: | `:oyster:` | | |

#### Aliment sucré

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#food--drink) | :icecream: | `:icecream:` | :shaved_ice: | `:shaved_ice:` |
| [top](#food--drink) | :ice_cream: | `:ice_cream:` | :doughnut: | `:doughnut:` |
| [top](#food--drink) | :cookie: | `:cookie:` | :birthday: | `:birthday:` |
| [top](#food--drink) | :cake: | `:cake:` | :cupcake: | `:cupcake:` |
| [top](#food--drink) | :pie: | `:pie:` | :chocolate_bar: | `:chocolate_bar:` |
| [top](#food--drink) | :candy: | `:candy:` | :lollipop: | `:lollipop:` |
| [top](#food--drink) | :custard: | `:custard:` | :honey_pot: | `:honey_pot:` |

#### Boisson

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#food--drink) | :baby_bottle: | `:baby_bottle:` | :milk_glass: | `:milk_glass:` |
| [top](#food--drink) | :coffee: | `:coffee:` | :teapot: | `:teapot:` |
| [top](#food--drink) | :tea: | `:tea:` | :sake: | `:sake:` |
| [top](#food--drink) | :champagne: | `:champagne:` | :wine_glass: | `:wine_glass:` |
| [top](#food--drink) | :cocktail: | `:cocktail:` | :tropical_drink: | `:tropical_drink:` |
| [top](#food--drink) | :beer: | `:beer:` | :beers: | `:beers:` |
| [top](#food--drink) | :clinking_glasses: | `:clinking_glasses:` | :tumbler_glass: | `:tumbler_glass:` |
| [top](#food--drink) | :cup_with_straw: | `:cup_with_straw:` | :bubble_tea: | `:bubble_tea:` |
| [top](#food--drink) | :beverage_box: | `:beverage_box:` | :mate: | `:mate:` |
| [top](#food--drink) | :ice_cube: | `:ice_cube:` | | |

#### Vaisselle

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#food--drink) | :chopsticks: | `:chopsticks:` | :plate_with_cutlery: | `:plate_with_cutlery:` |
| [top](#food--drink) | :fork_and_knife: | `:fork_and_knife:` | :spoon: | `:spoon:` |
| [top](#food--drink) | :hocho: | `:hocho:` <br /> `:knife:` | :amphora: | `:amphora:` |

### Voyages et lieux

#### Plan du lieu

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#travel--places) | :earth_africa: | `:earth_africa:` | :earth_americas: | `:earth_americas:` |
| [top](#travel--places) | :earth_asia: | `:earth_asia:` | :globe_with_meridians: | `:globe_with_meridians:` |
| [top](#travel--places) | :world_map: | `:world_map:` | :japan: | `:japan:` |
| [top](#travel--places) | :compass: | `:compass:` | | |

#### Lieu géographique

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#travel--places) | :mountain_snow: | `:mountain_snow:` | :mountain: | `:mountain:` |
| [top](#travel--places) | :volcano: | `:volcano:` | :mount_fuji: | `:mount_fuji:` |
| [top](#travel--places) | :camping: | `:camping:` | :beach_umbrella: | `:beach_umbrella:` |
| [top](#travel--places) | :desert: | `:desert:` | :desert_island: | `:desert_island:` |
| [top](#travel--places) | :national_park: | `:national_park:` | | |

#### Bâtiment

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#travel--places) | :stadium: | `:stadium:` | :classical_building: | `:classical_building:` |
| [top](#travel--places) | :building_construction: | `:building_construction:` | :bricks: | `:bricks:` |
| [top](#travel--places) | :rock: | `:rock:` | :wood: | `:wood:` |
| [top](#travel--places) | :hut: | `:hut:` | :houses: | `:houses:` |
| [top](#travel--places) | :derelict_house: | `:derelict_house:` | :house: | `:house:` |
| [top](#travel--places) | :house_with_garden: | `:house_with_garden:` | :office: | `:office:` |
| [top](#travel--places) | :post_office: | `:post_office:` | :european_post_office: | `:european_post_office:` |
| [top](#travel--places) | :hospital: | `:hospital:` | :bank: | `:bank:` |
| [top](#travel--places) | :hotel: | `:hotel:` | :love_hotel: | `:love_hotel:` |
| [top](#travel--places) | :convenience_store: | `:convenience_store:` | :school: | `:school:` |
| [top](#travel--places) | :department_store: | `:department_store:` | :factory: | `:factory:` |
| [top](#travel--places) | :japanese_castle: | `:japanese_castle:` | :european_castle: | `:european_castle:` |
| [top](#travel--places) | :wedding: | `:wedding:` | :tokyo_tower: | `:tokyo_tower:` |
| [top](#travel--places) | :statue_of_liberty: | `:statue_of_liberty:` | | |

#### Lieu religieux

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#travel--places) | :church: | `:church:` | :mosque: | `:mosque:` |
| [top](#travel--places) | :hindu_temple: | `:hindu_temple:` | :synagogue: | `:synagogue:` |
| [top](#travel--places) | :shinto_shrine: | `:shinto_shrine:` | :kaaba: | `:kaaba:` |

#### Lieu Autre

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#travel--places) | :fountain: | `:fountain:` | :tent: | `:tent:` |
| [top](#travel--places) | :foggy: | `:foggy:` | :night_with_stars: | `:night_with_stars:` |
| [top](#travel--places) | :cityscape: | `:cityscape:` | :sunrise_over_mountains: | `:sunrise_over_mountains:` |
| [top](#travel--places) | :sunrise: | `:sunrise:` | :city_sunset: | `:city_sunset:` |
| [top](#travel--places) | :city_sunrise: | `:city_sunrise:` | :bridge_at_night: | `:bridge_at_night:` |
| [top](#travel--places) | :hotsprings: | `:hotsprings:` | :carousel_horse: | `:carousel_horse:` |
| [top](#travel--places) | :ferris_wheel: | `:ferris_wheel:` | :roller_coaster: | `:roller_coaster:` |
| [top](#travel--places) | :barber: | `:barber:` | :circus_tent: | `:circus_tent:` |

#### Transport terrestre

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#travel--places) | :steam_locomotive: | `:steam_locomotive:` | :railway_car: | `:railway_car:` |
| [top](#travel--places) | :bullettrain_side: | `:bullettrain_side:` | :bullettrain_front: | `:bullettrain_front:` |
| [top](#travel--places) | :train2: | `:train2:` | :metro: | `:metro:` |
| [top](#travel--places) | :light_rail: | `:light_rail:` | :station: | `:station:` |
| [top](#travel--places) | :tram: | `:tram:` | :monorail: | `:monorail:` |
| [top](#travel--places) | :mountain_railway: | `:mountain_railway:` | :train: | `:train:` |
| [top](#travel--places) | :bus: | `:bus:` | :oncoming_bus: | `:oncoming_bus:` |
| [top](#travel--places) | :trolleybus: | `:trolleybus:` | :minibus: | `:minibus:` |
| [top](#travel--places) | :ambulance: | `:ambulance:` | :fire_engine: | `:fire_engine:` |
| [top](#travel--places) | :police_car: | `:police_car:` | :oncoming_police_car: | `:oncoming_police_car:` |
| [top](#travel--places) | :taxi: | `:taxi:` | :oncoming_taxi: | `:oncoming_taxi:` |
| [top](#travel--places) | :car: | `:car:` <br /> `:red_car:` | :oncoming_automobile: | `:oncoming_automobile:` |
| [top](#travel--places) | :blue_car: | `:blue_car:` | :pickup_truck: | `:pickup_truck:` |
| [top](#travel--places) | :truck: | `:truck:` | :articulated_lorry: | `:articulated_lorry:` |
| [top](#travel--places) | :tractor: | `:tractor:` | :racing_car: | `:racing_car:` |
| [top](#travel--places) | :motorcycle: | `:motorcycle:` | :motor_scooter: | `:motor_scooter:` |
| [top](#travel--places) | :manual_wheelchair: | `:manual_wheelchair:` | :motorized_wheelchair: | `:motorized_wheelchair:` |
| [top](#travel--places) | :auto_rickshaw: | `:auto_rickshaw:` | :bike: | `:bike:` |
| [top](#travel--places) | :kick_scooter: | `:kick_scooter:` | :skateboard: | `:skateboard:` |
| [top](#travel--places) | :roller_skate: | `:roller_skate:` | :busstop: | `:busstop:` |
| [top](#travel--places) | :motorway: | `:motorway:` | :railway_track: | `:railway_track:` |
| [top](#travel--places) | :oil_drum: | `:oil_drum:` | :fuelpump: | `:fuelpump:` |
| [top](#travel--places) | :rotating_light: | `:rotating_light:` | :traffic_light: | `:traffic_light:` |
| [top](#travel--places) | :vertical_traffic_light: | `:vertical_traffic_light:` | :stop_sign: | `:stop_sign:` |
| [top](#travel--places) | :construction: | `:construction:` | | |

#### Transport Eau

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#travel--places) | :anchor: | `:anchor:` | :boat: | `:boat:` <br /> `:sailboat:` |
| [top](#travel--places) | :canoe: | `:canoe:` | :speedboat: | `:speedboat:` |
| [top](#travel--places) | :passenger_ship: | `:passenger_ship:` | :ferry: | `:ferry:` |
| [top](#travel--places) | :motor_boat: | `:motor_boat:` | :ship: | `:ship:` |

#### Transport aérien

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#travel--places) | :airplane: | `:airplane:` | :small_airplane: | `:small_airplane:` |
| [top](#travel--places) | :flight_departure: | `:flight_departure:` | :flight_arrival: | `:flight_arrival:` |
| [top](#travel--places) | :parachute: | `:parachute:` | :seat: | `:seat:` |
| [top](#travel--places) | :helicopter: | `:helicopter:` | :suspension_railway: | `:suspension_railway:` |
| [top](#travel--places) | :mountain_cableway: | `:mountain_cableway:` | :aerial_tramway: | `:aerial_tramway:` |
| [top](#travel--places) | :artificial_satellite: | `:artificial_satellite:` | :rocket: | `:rocket:` |
| [top](#travel--places) | :flying_saucer: | `:flying_saucer:` | | |

#### Hôtel

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#travel--places) | :bellhop_bell: | `:bellhop_bell:` | :luggage: | `:luggage:` |

#### Heure

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#travel--places) | :hourglass: | `:hourglass:` | :hourglass_flowing_sand: | `:hourglass_flowing_sand:` |
| [top](#travel--places) | :watch: | `:watch:` | :alarm_clock: | `:alarm_clock:` |
| [top](#travel--places) | :stopwatch: | `:stopwatch:` | :timer_clock: | `:timer_clock:` |
| [top](#travel--places) | :mantelpiece_clock: | `:mantelpiece_clock:` | :clock12: | `:clock12:` |
| [top](#travel--places) | :clock1230: | `:clock1230:` | :clock1: | `:clock1:` |
| [top](#travel--places) | :clock130: | `:clock130:` | :clock2: | `:clock2:` |
| [top](#travel--places) | :clock230: | `:clock230:` | :clock3: | `:clock3:` |
| [top](#travel--places) | :clock330: | `:clock330:` | :clock4: | `:clock4:` |
| [top](#travel--places) | :clock430: | `:clock430:` | :clock5: | `:clock5:` |
| [top](#travel--places) | :clock530: | `:clock530:` | :clock6: | `:clock6:` |
| [top](#travel--places) | :clock630: | `:clock630:` | :clock7: | `:clock7:` |
| [top](#travel--places) | :clock730: | `:clock730:` | :clock8: | `:clock8:` |
| [top](#travel--places) | :clock830: | `:clock830:` | :clock9: | `:clock9:` |
| [top](#travel--places) | :clock930: | `:clock930:` | :clock10: | `:clock10:` |
| [top](#travel--places) | :clock1030: | `:clock1030:` | :clock11: | `:clock11:` |
| [top](#travel--places) | :clock1130: | `:clock1130:` | | |

#### Ciel et météo

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| [top](#travel--places) | :new_moon: | `:new_moon:` | :waxing_crescent_moon: | `:waxing_crescent_moon:` |
| [top](#travel--places) | :first_quarter_moon: | `:first_quarter_moon:` | :moon: | `:moon:` <br /> `:waxing_gibbous_moon:` |
| [top](#travel--places) | :full_moon: | `:full_moon:` | :waning_gibbous_moon: | `:waning_gibbous_moon:` |
| [top](#travel--places) | :last_quarter_moon: | `:last_quarter_moon:` | :waning_crescent_moon: | `:waning_crescent_moon:` |
| [top](#travel--places) | :crescent_moon: | `:crescent_moon:` | :new_moon_with_face: | `:new_moon_with_face:` |
| [top](#travel--places) | :first_quarter_moon_with_face: | `:first_quarter_moon_with_face:` | :last_quarter_moon_with_face: | `:last_quarter_moon_with_face:` |
| [top](#travel--places) | :thermometer: | `:thermometer:` | :sunny: | `:sunny:` |
| [top](#travel--places) | :full_moon_with_face: | `:full_moon_with_face:` | :sun_with_face: | `:sun_with_face:` |
| [top](#travel--places) | :ringed_planet: | `:ringed_planet:` | :star: | `:star:` |
| [top](#travel--places) | :star2: | `:star2:` | :stars: | `:stars:` |
| [top](#travel--places) | :milky_way: | `:milky_way:` | :cloud: | `:cloud:` |
| [top](#travel--places) | :partly_sunny: | `:partly_sunny:` | :cloud_with_lightning_and_rain: | `:cloud_with_lightning_and_rain:` |
| [top](#travel--places) | :sun_behind_small_cloud: | `:sun_behind_small_cloud:` | :sun_behind_large_cloud: | `:sun_behind_large_cloud:` |
| [top](#travel--places) | :sun_behind_rain_cloud: | `:sun_behind_rain_cloud:` | :cloud_with_rain: | `:cloud_with_rain:` |
| [top](#travel--places) | :cloud_with_snow: | `:cloud_with_snow:` | :cloud_with_lightning: | `:cloud_with_lightning:` |
| [top](#travel--places) | :tornado: | `:tornado:` | :fog: | `:fog:` |
| [top](#travel--places) | :wind_face: | `:wind_face:` | :cyclone: | `:cyclone:` |
| [top](#travel--places) | :rainbow: | `:rainbow:` | :closed_umbrella: | `:closed_umbrella:` |
| [top](#travel--places) | :open_umbrella: | `:open_umbrella:` | :umbrella: | `:umbrella:` |
| [top](#travel--places) | :parasol_on_ground: | `:parasol_on_ground:` | :zap: | `:zap:` |
| [top](#travel--places) | :snowflake: | `:snowflake:` | :snowman_with_snow: | `:snowman_with_snow:` |
| [top](#travel--places) | :snowman: | `:snowman:` | :comet: | `:comet:` |
| [top](#travel--places) | :fire: | `:fire:` | :droplet: | `:droplet:` |
| [top](#travel--places) | :ocean: | `:ocean:` | | |

### Activités

#### Événement

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :jack_o_lantern: | `:jack_o_lantern:` | :christmas_tree: | `:christmas_tree:` |
| :fireworks: | `:fireworks:` | :sparkler: | `:sparkler:` |
| :firecracker: | `:firecracker:` | :sparkles: | `:sparkles:` |
| :balloon: | `:balloon:` | :tada: | `:tada:` |
| :confetti_ball: | `:confetti_ball:` | :tanabata_tree: | `:tanabata_tree:` |
| :bamboo: | `:bamboo:` | :dolls: | `:dolls:` |
| :flags: | `:flags:` | :wind_chime: | `:wind_chime:` |
| :rice_scene: | `:rice_scene:` | :red_envelope: | `:red_envelope:` |
| :ribbon: | `:ribbon:` | :gift: | `:gift:` |
| :reminder_ribbon: | `:reminder_ribbon:` | :tickets: | `:tickets:` |
| :ticket: | `:ticket:` | | |

#### Médaille

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :medal_military: | `:medal_military:` | :trophy: | `:trophy:` |
| :medal_sports: | `:medal_sports:` | :1st_place_medal: | `:1st_place_medal:` |
| :2nd_place_medal: | `:2nd_place_medal:` | :3rd_place_medal: | `:3rd_place_medal:` |

#### Sport

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :soccer: | `:soccer:` | :baseball: | `:baseball:` |
| :softball: | `:softball:` | :basketball: | `:basketball:` |
| :volleyball: | `:volleyball:` | :football: | `:football:` |
| :rugby_football: | `:rugby_football:` | :tennis: | `:tennis:` |
| :flying_disc: | `:flying_disc:` | :bowling: | `:bowling:` |
| :cricket_game: | `:cricket_game:` | :field_hockey: | `:field_hockey:` |
| :ice_hockey: | `:ice_hockey:` | :lacrosse: | `:lacrosse:` |
| :ping_pong: | `:ping_pong:` | :badminton: | `:badminton:` |
| :boxing_glove: | `:boxing_glove:` | :martial_arts_uniform: | `:martial_arts_uniform:` |
| :goal_net: | `:goal_net:` | :golf: | `:golf:` |
| :ice_skate: | `:ice_skate:` | :fishing_pole_and_fish: | `:fishing_pole_and_fish:` |
| :diving_mask: | `:diving_mask:` | :running_shirt_with_sash: | `:running_shirt_with_sash:` |
| :ski: | `:ski:` | :sled: | `:sled:` |
| :curling_stone: | `:curling_stone:` | | |

#### Jeu

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :dart: | `:dart:` | :yo_yo: | `:yo_yo:` |
| :kite: | `:kite:` | :gun: | `:gun:` |
| :8ball: | `:8ball:` | :crystal_ball: | `:crystal_ball:` |
| :magic_wand: | `:magic_wand:` | :video_game: | `:video_game:` |
| :joystick: | `:joystick:` | :slot_machine: | `:slot_machine:` |
| :game_die: | `:game_die:` | :jigsaw: | `:jigsaw:` |
| :teddy_bear: | `:teddy_bear:` | :pinata: | `:pinata:` |
| :nesting_dolls: | `:nesting_dolls:` | :spades: | `:spades:` |
| :hearts: | `:hearts:` | :diamonds: | `:diamonds:` |
| :clubs: | `:clubs:` | :chess_pawn: | `:chess_pawn:` |
| :black_joker: | `:black_joker:` | :mahjong: | `:mahjong:` |
| :flower_playing_cards: | `:flower_playing_cards:` | | |

#### Artisanat

| ico | shortcode | ico | shortcode |
| :-: | - | :-: | - |
| :performing_arts: | `:performing_arts:` | :framed_picture: | `:framed_picture:` |
| :art: | `:art:` | :thread: | `:thread:` |
| :sewing_needle: | `:sewing_needle:` | :yarn: | `:yarn:` |
| :knot: | `:knot:` | | |

