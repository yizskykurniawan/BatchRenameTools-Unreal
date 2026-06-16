## Batch Rename Tools for Unreal Engine 5

The framework uses the unreal Python module to talk to the engine's asset registry. This allows for "Safe Renaming," which means that even if a file is renamed, the materials and animations attached to it will not break.

​The results show that as the number of assets grows, the automated tool saves an increasing amount of time. For large scenes with 500 assets, the tool completes in seconds, while manual naming takes over an hour.

### Basic Script (Use in Unreal)
Main Script for Batch Rename.
```ruby
import unreal

selected_assets = unreal.EditorUtilityLibrary.get_selected_assets()

# This important component to run bacth rename in unreal to get each naming by index, use Export Naming Mesh Script for Maya
new_names = [
    "00_SM_P_pSphere1",
    "01_SM_P_pCube1",
    "02_SM_P_pCylinder1",
    "03_SM_P_pCone1",
    "04_SM_P_pTorus1",
    "05_SM_P_pPlane1",
    "06_SM_P_pDisc1"
]

for i, asset in enumerate(selected_assets):
    if i < len(new_names):
        new_name = new_names[i]
        old_path = asset.get_path_name()
        folder = old_path.rsplit('/', 1)[0]
        new_path = f"{folder}/{new_name}"
        unreal.EditorAssetLibrary.rename_asset(old_path, new_path)
        unreal.log(f"Asset {old_path} change log {new_name}")
```
### Batch Rename UE v2.0.1 (Use in Maya) - New Method
Newer Version, Auto copy into new names. Try it below!
```ruby
import maya.cmds as cmds
from PySide2 import QtWidgets, QtCore, QtGui

# ==========================================
# LOGIC (STATIC SET)
# ==========================================
def generate_full_unreal_script(obj_list, start_zero=True, prefix="SM_P"):
    """Creates the entire Unreal Python script as a single string."""
    start_index = 0 if start_zero else 1
    
    unreal_script = [
        "import unreal",
        "",
        "selected_assets = unreal.EditorUtilityLibrary.get_selected_assets()",
        "",
        "# Generated from Maya Export Tool",
        "new_names = ["
    ]
    
    for i, obj in enumerate(obj_list):
        idx = i + start_index
        name_str = f"{idx:02d}_{prefix}_{obj}"
        unreal_script.append(f'    "{name_str}",')
    
    unreal_script.extend([
        "]",
        "",
        "for i, asset in enumerate(selected_assets):",
        "    if i < len(new_names):",
        "        new_name = new_names[i]",
        "        old_path = asset.get_path_name()",
        "        folder = old_path.rsplit('/', 1)[0]",
        "        new_path = f'{folder}/{new_name}'",
        "        ",
        "        if old_path != new_path:",
        "            unreal.EditorAssetLibrary.rename_asset(old_path, new_path)",
        "            print(f'Renamed: {new_name}')",
        "    else:",
        "        print('Warning: More assets selected than names provided.')"
    ])
    
    return "\n".join(unreal_script)

# ==========================================
# LOGIC (SKELETAL SET)
# ==========================================
def generate_skeletal_set_script(nameAsset):
    """Creates the specific SKM/AS/PHYS/SKEL script."""
    unreal_script = [
        "import unreal",
        "",
        "selected_assets = unreal.EditorUtilityLibrary.get_selected_assets()",
        "",
        f"# Generated for Skeletal Set: {nameAsset}",
        "new_names = [",
        f'    "00_SKM_P_{nameAsset}",',
        f'    "01_AS_P_{nameAsset}",',
        f'    "02_PHYS_P_{nameAsset}",',
        f'    "03_SKEL_P_{nameAsset}"',
        "]",
        "",
        "for i, asset in enumerate(selected_assets):",
        "    if i < len(new_names):",
        "        new_name = new_names[i]",
        "        old_path = asset.get_path_name()",
        "        folder = old_path.rsplit('/', 1)[0]",
        "        new_path = f'{folder}/{new_name}'",
        "        if old_path != new_path:",
        "            unreal.EditorAssetLibrary.rename_asset(old_path, new_path)",
        "            print(f'Renamed: {new_name}')",
        "    else:",
        "        print('Warning: More assets selected than names provided.')"
    ]
    return "\n".join(unreal_script)

class UnrealScriptGenerator(QtWidgets.QWidget):
    def __init__(self):
        super(UnrealScriptGenerator, self).__init__()
        self.setWindowTitle("Batch Rename 2.0")
        self.setFixedSize(320, 560)
        
        layout = QtWidgets.QVBoxLayout(self)

        # --- OLD SETTINGS SECTION ---
        layout.addWidget(QtWidgets.QLabel("<b>General Settings:</b>"))
        self.startZeroCB = QtWidgets.QCheckBox("Start Index from 00")
        self.startZeroCB.setChecked(True)
        layout.addWidget(self.startZeroCB)

        layout.addSpacing(5)
        layout.addWidget(QtWidgets.QLabel("Custom Prefix:"))
        prefix_layout = QtWidgets.QHBoxLayout()
        self.prefixEdit = QtWidgets.QLineEdit("SM_P")
        self.btn_reset = QtWidgets.QPushButton("Reset")
        self.btn_reset.setFixedWidth(50)
        self.btn_reset.clicked.connect(lambda: self.prefixEdit.setText("SM_P"))
        prefix_layout.addWidget(self.prefixEdit)
        prefix_layout.addWidget(self.btn_reset)
        layout.addLayout(prefix_layout)

        layout.addSpacing(15)
        line = QtWidgets.QFrame(); line.setFrameShape(QtWidgets.QFrame.HLine)
        layout.addWidget(line)

        # --- NEW MANUAL INPUT SECTION ---
        layout.addSpacing(5)
        layout.addWidget(QtWidgets.QLabel("<b>Asset Name (for Skeletal Set):</b>"))
        self.manualNameEdit = QtWidgets.QLineEdit("-")
        layout.addWidget(self.manualNameEdit)
        layout.addSpacing(15)
        line2 = QtWidgets.QFrame(); line2.setFrameShape(QtWidgets.QFrame.HLine)
        layout.addWidget(line2)

        # --- ACTIONS SECTION ---
        layout.addWidget(QtWidgets.QLabel("<b>Export Actions:</b>"))
        
        # Static Buttons
        self.btn_mesh = QtWidgets.QPushButton("COPY STATIC SET SCRIPT\n(Selected Meshes)")
        self.btn_mesh.setMinimumHeight(55)
        self.btn_mesh.setStyleSheet("background-color: #354a3b; font-weight: bold; color: #e0e0e0;")
        self.btn_mesh.clicked.connect(lambda: self.execute_copy("mesh"))
        layout.addWidget(self.btn_mesh)

        self.btn_grp = QtWidgets.QPushButton("COPY STATIC SET SCRIPT\n(Selected Groups)")
        self.btn_grp.setMinimumHeight(55)
        self.btn_grp.setStyleSheet("font-weight: bold;")
        self.btn_grp.clicked.connect(lambda: self.execute_copy("group"))
        layout.addWidget(self.btn_grp)

        # Skeletal Buttons
        self.btn_skel = QtWidgets.QPushButton("COPY SKELETAL SET SCRIPT\n(SKM, AS, PHYS, SKEL)")
        self.btn_skel.setMinimumHeight(55)
        self.btn_skel.setStyleSheet("background-color: #4a3c35; font-weight: bold; color: #ffe0b3;")
        self.btn_skel.clicked.connect(self.execute_skeletal_copy)
        layout.addWidget(self.btn_skel)
        
        layout.addStretch()
        
        footer = QtWidgets.QLabel("Batch Rename to Unreal Engine")
        footer.setStyleSheet("font-size: 9px; color: #888;")
        footer.setAlignment(QtCore.Qt.AlignCenter)
        layout.addWidget(footer)

    def execute_copy(self, mode):
        # Old Logic
        items = []
        if mode == "mesh":
            sel = cmds.ls(selection=True, dag=True, type="mesh")
            if sel:
                items = cmds.listRelatives(sel, parent=True, fullPath=False)
        else:
            sel = cmds.ls(selection=True, type="transform")
            items = [t for t in sel if not cmds.listRelatives(t, shapes=True)]

        if not items:
            QtWidgets.QMessageBox.warning(self, "Selection Error", f"No {mode} items selected!")
            return

        full_code = generate_full_unreal_script(
            items, 
            self.startZeroCB.isChecked(), 
            self.prefixEdit.text()
        )
        self._to_clipboard(full_code, len(items))

    def execute_skeletal_copy(self):
        # New logic to handle the manual name input
        asset_name = self.manualNameEdit.text().strip()
        if not asset_name:
            QtWidgets.QMessageBox.warning(self, "Error", "Please enter a name in the box!")
            return
            
        full_code = generate_skeletal_set_script(asset_name)
        self._to_clipboard(full_code, 4)

    def _to_clipboard(self, text, count):
        clipboard = QtWidgets.QApplication.clipboard()
        clipboard.setText(text)
        QtWidgets.QMessageBox.information(self, "Success", f"Script for {count} items copied!")

def show_ui():
    global mayaToUnrealWin
    try:
        mayaToUnrealWin.close()
        mayaToUnrealWin.deleteLater()
    except:
        pass
    mayaToUnrealWin = UnrealScriptGenerator()
    mayaToUnrealWin.show()

show_ui()
```
### Comparison Effectivity
Between each other method for Renaming Static Mesh in Unreal.
<img width="1318" height="762" alt="image" src="https://github.com/user-attachments/assets/894f48e6-bc50-4bd9-b3b4-a313b13399dd" />

All code and flow in this page, iterate by Yizsky Kurniawan.

License Common Attribute. 
Can used for commercial and personal used. 



