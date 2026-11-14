extends Node3D
# RANDY: School of Secrets - Lorong lantai 1 (tekstur prosedural, pintu, jendela, lampu berkedip)
var player: CharacterBody3D
var prompt: Label
var btn: Button
var journal: Label
var found: Array = []
var target: Node3D = null
var flicker: Array = []
var title_layer: CanvasLayer

var m_wall: StandardMaterial3D
var m_band: StandardMaterial3D
var m_floor: StandardMaterial3D
var m_ceil: StandardMaterial3D
var m_wood: StandardMaterial3D
var m_frame: StandardMaterial3D
var m_metal: StandardMaterial3D
var m_locker: StandardMaterial3D
var m_board: StandardMaterial3D
var m_paper: StandardMaterial3D
var m_glass: StandardMaterial3D
var m_tube: StandardMaterial3D

const EVIDENCE := [
	{"name": "Catatan Murid", "desc": "Tulisan tangan: 'Jangan sendirian di ruang agama.'", "pos": Vector3(1.4, 0.78, -8)},
	{"name": "Surat Robek", "desc": "Surat dari murid kelas 11 kepada wali kelas.", "pos": Vector3(-1.4, 0.78, -16)},
	{"name": "Foto Lama", "desc": "Foto kegiatan dengan tanggal yang dicoret.", "pos": Vector3(1.4, 0.78, -24)},
]
const ROOMS := ["X-1", "X-2", "XI-1", "XI-2", "RUANG BK", "UKS"]

func _ready() -> void:
	_make_materials()
	_build_env()
	_build_corridor()
	_build_doors()
	_build_windows()
	_build_props()
	_build_lights()
	_build_dust()
	_build_evidence()
	player = preload("res://player.gd").new()
	player.position = Vector3(0, 0.05, 2)
	add_child(player)
	player.set_process_unhandled_input(false)
	_build_ui()
	_build_title()

# ---------- tekstur & material ----------
func _tex(base: Color, noise_amt: float, grid: int, grid_col: Color, band_dark := false) -> ImageTexture:
	var size := 128
	var img := Image.create(size, size, false, Image.FORMAT_RGB8)
	var rng := RandomNumberGenerator.new()
	rng.seed = hash(str(base))
	var fn := FastNoiseLite.new()
	fn.seed = int(rng.seed % 100000)
	fn.frequency = 0.04
	var cell := 1
	if grid > 0:
		cell = int(size / grid)
	for y in range(size):
		for x in range(size):
			var n: float = fn.get_noise_2d(x, y) * noise_amt + rng.randf_range(-0.02, 0.02)
			var c := Color(base.r + n, base.g + n, base.b + n)
			if grid > 0 and (x % cell < 2 or y % cell < 2):
				c = grid_col
			img.set_pixel(x, y, c)
	img.generate_mipmaps()
	return ImageTexture.create_from_image(img)

func _pmat(tex: Texture2D, rough: float, uv: float, tint := Color(1, 1, 1)) -> StandardMaterial3D:
	var m := StandardMaterial3D.new()
	m.albedo_texture = tex
	m.albedo_color = tint
	m.roughness = rough
	m.uv1_triplanar = true
	m.uv1_world_triplanar = true
	m.uv1_scale = Vector3(uv, uv, uv)
	return m

func _mat(c: Color, rough := 0.8, metal := 0.0) -> StandardMaterial3D:
	var m := StandardMaterial3D.new()
	m.albedo_color = c
	m.roughness = rough
	m.metallic = metal
	return m

func _make_materials() -> void:
	m_wall = _pmat(_tex(Color(0.55, 0.57, 0.53), 0.12, 0, Color.BLACK), 0.95, 0.5)
	m_band = _pmat(_tex(Color(0.16, 0.26, 0.24), 0.08, 0, Color.BLACK), 0.7, 0.5)
	m_floor = _pmat(_tex(Color(0.20, 0.30, 0.30), 0.06, 4, Color(0.08, 0.12, 0.12)), 0.25, 0.5)
	m_ceil = _pmat(_tex(Color(0.40, 0.40, 0.38), 0.05, 2, Color(0.2, 0.2, 0.2)), 0.95, 0.5)
	m_wood = _pmat(_tex(Color(0.30, 0.18, 0.09), 0.10, 0, Color.BLACK), 0.6, 1.0)
	m_frame = _mat(Color(0.12, 0.12, 0.12), 0.7)
	m_metal = _mat(Color(0.65, 0.65, 0.68), 0.3, 0.8)
	m_locker = _mat(Color(0.35, 0.42, 0.48), 0.5, 0.4)
	m_board = _mat(Color(0.10, 0.28, 0.18), 0.9)
	m_paper = _mat(Color(0.92, 0.9, 0.8), 0.9)
	m_paper.emission_enabled = true
	m_paper.emission = Color(0.5, 0.45, 0.3)
	m_paper.emission_energy_multiplier = 0.6
	m_glass = _mat(Color(0.05, 0.08, 0.12), 0.1)
	m_glass.emission_enabled = true
	m_glass.emission = Color(0.3, 0.45, 0.65)
	m_glass.emission_energy_multiplier = 0.9
	m_tube = _mat(Color(1, 1, 1), 0.5)
	m_tube.emission_enabled = true
	m_tube.emission = Color(0.85, 0.95, 1.0)
	m_tube.emission_energy_multiplier = 2.0

func _box(pos: Vector3, size: Vector3, mat: Material, solid := true) -> void:
	var body := StaticBody3D.new()
	body.position = pos
	var mi := MeshInstance3D.new()
	var bm := BoxMesh.new()
	bm.size = size
	mi.mesh = bm
	mi.material_override = mat
	body.add_child(mi)
	if solid:
		var cs := CollisionShape3D.new()
		var sh := BoxShape3D.new()
		sh.size = size
		cs.shape = sh
		body.add_child(cs)
	add_child(body)

func _label3d(txt: String, pos: Vector3, rot_y: float, px := 0.004) -> void:
	var l := Label3D.new()
	l.text = txt
	l.font_size = 64
	l.pixel_size = px
	l.modulate = Color(0.9, 0.9, 0.8)
	l.position = pos
	l.rotation_degrees = Vector3(0, rot_y, 0)
	add_child(l)

# ---------- lingkungan ----------
func _build_env() -> void:
	var we := WorldEnvironment.new()
	var env := Environment.new()
	env.background_mode = Environment.BG_COLOR
	env.background_color = Color(0.01, 0.015, 0.02)
	env.ambient_light_source = Environment.AMBIENT_SOURCE_COLOR
	env.ambient_light_color = Color(0.22, 0.27, 0.32)
	env.ambient_light_energy = 0.4
	env.tonemap_mode = Environment.TONE_MAPPER_FILMIC
	env.fog_enabled = true
	env.fog_light_color = Color(0.06, 0.08, 0.1)
	env.fog_density = 0.035
	env.glow_enabled = true
	env.glow_intensity = 0.7
	env.glow_bloom = 0.08
	we.environment = env
	add_child(we)

func _build_corridor() -> void:
	_box(Vector3(0, -0.1, -15), Vector3(4, 0.2, 40), m_floor)
	_box(Vector3(0, 3.1, -15), Vector3(4, 0.2, 40), m_ceil)
	# dinding kiri/kanan: bagian atas plester + bagian bawah hijau tua
	for side in [-1, 1]:
		_box(Vector3(side * 2.1, 2.05, -15), Vector3(0.2, 1.9, 40), m_wall)
		_box(Vector3(side * 2.09, 0.55, -15), Vector3(0.24, 1.1, 40), m_band)
	_box(Vector3(0, 1.5, 5.1), Vector3(4, 3, 0.2), m_wall)
	_box(Vector3(0, 1.5, -35.1), Vector3(4, 3, 0.2), m_wall)
	# papan nama lantai di ujung
	_label3d("LANTAI 1 - TANGGA", Vector3(0, 2.3, -34.95), 0.0, 0.006)

func _build_doors() -> void:
	for i in range(ROOMS.size()):
		var z := -3.0 - i * 5.5
		_box(Vector3(-1.97, 1.1, z), Vector3(0.08, 2.25, 1.2), m_frame, false)
		_box(Vector3(-1.94, 1.05, z), Vector3(0.06, 2.1, 1.0), m_wood, false)
		_box(Vector3(-1.89, 1.0, z + 0.38), Vector3(0.05, 0.04, 0.16), m_metal, false)
		_label3d(ROOMS[i], Vector3(-1.92, 2.55, z), 90.0)

func _build_windows() -> void:
	for i in range(6):
		var z := -1.5 - i * 5.5
		_box(Vector3(1.97, 1.85, z), Vector3(0.04, 1.3, 1.9), m_glass, false)
		_box(Vector3(1.95, 1.85, z), Vector3(0.05, 1.3, 0.05), m_frame, false)
		_box(Vector3(1.95, 1.85, z - 0.95), Vector3(0.06, 1.38, 0.06), m_frame, false)
		_box(Vector3(1.95, 1.85, z + 0.95), Vector3(0.06, 1.38, 0.06), m_frame, false)
	for z in [-1.5, -12.5, -23.5]:
		var l := OmniLight3D.new()
		l.position = Vector3(1.2, 1.8, z)
		l.light_color = Color(0.4, 0.55, 0.85)
		l.light_energy = 0.5
		l.omni_range = 4.0
		add_child(l)

func _build_props() -> void:
	# papan pengumuman di dinding kiri
	_box(Vector3(-1.93, 1.75, -11), Vector3(0.05, 1.0, 1.8), m_board, false)
	var zs := [-11.6, -11.2, -10.8, -10.4, -11.0]
	var ys := [1.95, 1.9, 1.98, 1.85, 1.6]
	for i in range(zs.size()):
		_box(Vector3(-1.9, ys[i], zs[i]), Vector3(0.02, 0.3, 0.22), m_paper, false)
	# loker di dinding kanan, dekat ujung
	_box(Vector3(1.7, 0.95, -31.5), Vector3(0.5, 1.9, 2.6), m_locker)
	# bangku
	_box(Vector3(-1.5, 0.45, -5.5), Vector3(0.5, 0.06, 1.4), m_wood)
	_box(Vector3(-1.5, 0.22, -5.5), Vector3(0.45, 0.4, 1.3), m_frame, false)

func _build_lights() -> void:
	for i in range(6):
		var z := 1.0 - i * 6.0
		var l := OmniLight3D.new()
		l.position = Vector3(0, 2.7, z)
		l.light_color = Color(0.8, 0.95, 1.0)
		l.light_energy = 1.1
		l.omni_range = 9.0
		l.shadow_enabled = i < 2
		add_child(l)
		_box(Vector3(0, 2.96, z), Vector3(0.25, 0.05, 1.2), m_tube, false)
		if i == 2 or i == 4:
			flicker.append({"light": l, "base": 1.1, "timer": 0.0})

func _build_dust() -> void:
	var p := CPUParticles3D.new()
	p.position = Vector3(0, 1.5, -15)
	p.amount = 80
	p.lifetime = 10.0
	p.emission_shape = CPUParticles3D.EMISSION_SHAPE_BOX
	p.emission_box_extents = Vector3(1.8, 1.4, 18)
	p.gravity = Vector3(0, 0.01, 0)
	p.initial_velocity_min = 0.0
	p.initial_velocity_max = 0.05
	var sm := SphereMesh.new()
	sm.radius = 0.01
	sm.height = 0.02
	var dm := _mat(Color(0.8, 0.85, 0.9), 1.0)
	dm.emission_enabled = true
	dm.emission = Color(0.5, 0.55, 0.6)
	sm.material = dm
	p.mesh = sm
	add_child(p)

func _build_evidence() -> void:
	for d in EVIDENCE:
		var p: Vector3 = d["pos"]
		# meja kecil supaya bukti tidak melayang
		_box(Vector3(p.x, 0.72, p.z), Vector3(0.8, 0.05, 0.5), m_wood)
		for lx in [-0.35, 0.35]:
			for lz in [-0.2, 0.2]:
				_box(Vector3(p.x + lx, 0.35, p.z + lz), Vector3(0.05, 0.7, 0.05), m_frame, false)
		var n := Node3D.new()
		n.position = p
		n.set_meta("name", d["name"])
		n.set_meta("desc", d["desc"])
		n.add_to_group("evidence")
		var mi := MeshInstance3D.new()
		var bm := BoxMesh.new()
		bm.size = Vector3(0.3, 0.02, 0.22)
		mi.mesh = bm
		mi.material_override = m_paper
		n.add_child(mi)
		add_child(n)

# ---------- UI ----------
func _build_ui() -> void:
	var layer := CanvasLayer.new()
	add_child(layer)
	player.attach_stick(layer)
	var cross := ColorRect.new()
	cross.color = Color(1, 1, 1, 0.7)
	cross.size = Vector2(6, 6)
	cross.set_anchors_and_offsets_preset(Control.PRESET_CENTER)
	cross.mouse_filter = Control.MOUSE_FILTER_IGNORE
	layer.add_child(cross)
	prompt = Label.new()
	prompt.set_anchors_and_offsets_preset(Control.PRESET_CENTER_BOTTOM)
	prompt.position.y -= 120
	prompt.add_theme_font_size_override("font_size", 28)
	prompt.mouse_filter = Control.MOUSE_FILTER_IGNORE
	layer.add_child(prompt)
	btn = Button.new()
	btn.text = "PERIKSA"
	btn.custom_minimum_size = Vector2(190, 100)
	btn.set_anchors_and_offsets_preset(Control.PRESET_BOTTOM_RIGHT)
	btn.position -= Vector2(210, 120)
	btn.add_theme_font_size_override("font_size", 28)
	btn.pressed.connect(_interact)
	layer.add_child(btn)
	journal = Label.new()
	journal.position = Vector2(20, 20)
	journal.add_theme_font_size_override("font_size", 22)
	journal.mouse_filter = Control.MOUSE_FILTER_IGNORE
	layer.add_child(journal)
	var floor_l := Label.new()
	floor_l.text = "LANTAI 1"
	floor_l.set_anchors_and_offsets_preset(Control.PRESET_TOP_RIGHT)
	floor_l.offset_left = -170
	floor_l.offset_top = 20
	floor_l.add_theme_font_size_override("font_size", 24)
	floor_l.modulate = Color(0.3, 0.7, 1.0)
	floor_l.mouse_filter = Control.MOUSE_FILTER_IGNORE
	layer.add_child(floor_l)
	var credit := Label.new()
	credit.text = "DieeDev"
	credit.set_anchors_and_offsets_preset(Control.PRESET_BOTTOM_LEFT)
	credit.offset_left = 20
	credit.offset_top = -40
	credit.modulate = Color(0.3, 0.7, 1.0, 0.7)
	credit.mouse_filter = Control.MOUSE_FILTER_IGNORE
	layer.add_child(credit)
	_refresh_journal()

func _build_title() -> void:
	title_layer = CanvasLayer.new()
	title_layer.layer = 10
	add_child(title_layer)
	var bg := ColorRect.new()
	bg.color = Color(0.02, 0.03, 0.05, 1.0)
	bg.set_anchors_and_offsets_preset(Control.PRESET_FULL_RECT)
	title_layer.add_child(bg)
	var cc := CenterContainer.new()
	cc.set_anchors_and_offsets_preset(Control.PRESET_FULL_RECT)
	bg.add_child(cc)
	var vb := VBoxContainer.new()
	vb.add_theme_constant_override("separation", 14)
	cc.add_child(vb)
	var t1 := Label.new()
	t1.text = "RANDY"
	t1.horizontal_alignment = HORIZONTAL_ALIGNMENT_CENTER
	t1.add_theme_font_size_override("font_size", 72)
	t1.modulate = Color(0.3, 0.7, 1.0)
	vb.add_child(t1)
	var t2 := Label.new()
	t2.text = "SCHOOL OF SECRETS"
	t2.horizontal_alignment = HORIZONTAL_ALIGNMENT_CENTER
	t2.add_theme_font_size_override("font_size", 32)
	vb.add_child(t2)
	var hint := Label.new()
	hint.text = "Joystick kiri = jalan   |   Geser kanan = lihat sekeliling\nDekati bukti, arahkan crosshair, lalu tekan PERIKSA"
	hint.horizontal_alignment = HORIZONTAL_ALIGNMENT_CENTER
	hint.add_theme_font_size_override("font_size", 20)
	hint.modulate = Color(0.7, 0.7, 0.7)
	vb.add_child(hint)
	var start := Button.new()
	start.text = "MULAI"
	start.custom_minimum_size = Vector2(260, 90)
	start.add_theme_font_size_override("font_size", 30)
	start.pressed.connect(_start_game)
	vb.add_child(start)
	var dev := Label.new()
	dev.text = "DieeDev"
	dev.horizontal_alignment = HORIZONTAL_ALIGNMENT_CENTER
	dev.modulate = Color(0.3, 0.7, 1.0, 0.7)
	vb.add_child(dev)

func _start_game() -> void:
	title_layer.queue_free()
	player.set_process_unhandled_input(true)

# ---------- gameplay ----------
func _process(delta: float) -> void:
	for e in flicker:
		e["timer"] = float(e["timer"]) - delta
		var lvl := 1.0
		if float(e["timer"]) > 0.0:
			lvl = randf_range(0.0, 0.5)
		elif randf() < 0.004:
			e["timer"] = randf_range(0.1, 0.4)
		(e["light"] as OmniLight3D).light_energy = float(e["base"]) * lvl
	target = null
	var cam: Camera3D = player.cam
	for n in get_tree().get_nodes_in_group("evidence"):
		var aim: Vector3 = n.global_position + Vector3(0, 0.5, 0)
		var to: Vector3 = aim - cam.global_position
		var dist: float = (n.global_position - cam.global_position).length()
		if dist < 2.6 and (-cam.global_transform.basis.z).dot(to.normalized()) > 0.65:
			target = n
			break
	prompt.text = "Tekan untuk memeriksa" if target else ""

func _interact() -> void:
	if target == null:
		return
	found.append(str(target.get_meta("name")) + ": " + str(target.get_meta("desc")))
	target.remove_from_group("evidence")
	target.queue_free()
	_refresh_journal()

func _refresh_journal() -> void:
	var t := "JURNAL (%d/%d bukti)" % [found.size(), EVIDENCE.size()]
	for f in found:
		t += "\n- " + f
	if found.size() >= EVIDENCE.size():
		t += "\n\nSemua bukti lorong terkumpul. Lantai 2 segera dibuka."
	journal.text = t
