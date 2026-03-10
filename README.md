## Batch Rename Tools for Unreal Engine 5

This script based Python Language to escalate renaming process using Maya to Unreal.
By using index outliner maya and then export each naming mesh, apply this script by selected first and end index.

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
### Batch Rename UE v1.0 (Use in Maya) - Old Method
Second Script for Export Naming Mesh.
```ruby
import maya.cmds as cmds
from PySide2 import QtWidgets, QtCore, QtGui

def format_line(i, obj, start_zero=True, fmt="Index - Name"):
    start_index = 0 if start_zero else 1
    idx = i + start_index

    if fmt == "Index - Name":
        return f"{idx:02d} - {obj}"
    elif fmt == "Name (Index)":
        return f"{obj} ({idx:02d})"
    elif fmt == "Index_SM_P_Name":
        return f"{idx:02d}_SM_P_{obj}"
    else:
        return f"{idx:02d} - {obj}"  # default


def export_names(obj_list, file_path=None, to_clipboard=True, start_zero=True, fmt="Index - Name"):
    lines = [format_line(i, obj, start_zero, fmt) for i, obj in enumerate(obj_list, start=0)]

    # Save to file if path is provided
    if file_path:
        with open(file_path, "w") as f:
            f.write("\n".join(lines))

    # Copy to clipboard via Qt
    if to_clipboard:
        clipboard = QtWidgets.QApplication.clipboard()
        clipboard.setText("\n".join(lines))

    msg = "Names exported successfully."
    if file_path:
        msg += f"\nFile: {file_path}"
    if to_clipboard:
        msg += "\nData also copied to clipboard."

    QtWidgets.QMessageBox.information(None, "Export Complete", msg)


class MeshExportUI(QtWidgets.QWidget):
    def __init__(self):
        super(MeshExportUI, self).__init__()
        self.setWindowTitle("Export Mesh/Group Names")
        self.setFixedSize(300, 420)

        layout = QtWidgets.QVBoxLayout(self)

        # Checkbox
        self.startZeroCB = QtWidgets.QCheckBox("Start index from 0 (uncheck = start from 1)")
        self.startZeroCB.setChecked(True)
        layout.addWidget(self.startZeroCB)

        # Dropdown
        self.formatMenu = QtWidgets.QComboBox()
        self.formatMenu.addItems(["Index - Name", "Name (Index)", "Index_SM_P_Name"])
        layout.addWidget(QtWidgets.QLabel("Output Format:"))
        layout.addWidget(self.formatMenu)

        # Mesh section
        layout.addWidget(QtWidgets.QLabel("Meshes:"))
        btn_all = QtWidgets.QPushButton("Export All Meshes")
        btn_all.clicked.connect(self.export_all_mesh)
        layout.addWidget(btn_all)

        btn_sel = QtWidgets.QPushButton("Export Selected Meshes")
        btn_sel.clicked.connect(self.export_selected_mesh)
        layout.addWidget(btn_sel)

        btn_clip_all = QtWidgets.QPushButton("Copy All Meshes to Clipboard")
        btn_clip_all.clicked.connect(self.copy_clipboard_only)
        layout.addWidget(btn_clip_all)

        btn_clip_sel = QtWidgets.QPushButton("Copy Selected Meshes to Clipboard")
        btn_clip_sel.clicked.connect(self.copy_selected_clipboard_only)
        layout.addWidget(btn_clip_sel)

        # Group section
        layout.addWidget(QtWidgets.QLabel("Groups:"))
        btn_grp_all = QtWidgets.QPushButton("Export All Groups")
        btn_grp_all.clicked.connect(self.export_all_groups)
        layout.addWidget(btn_grp_all)

        btn_grp_sel = QtWidgets.QPushButton("Export Selected Groups")
        btn_grp_sel.clicked.connect(self.export_selected_groups)
        layout.addWidget(btn_grp_sel)

        btn_grp_clip_all = QtWidgets.QPushButton("Copy All Groups to Clipboard")
        btn_grp_clip_all.clicked.connect(self.copy_groups_clipboard_only)
        layout.addWidget(btn_grp_clip_all)

        btn_grp_clip_sel = QtWidgets.QPushButton("Copy Selected Groups to Clipboard")
        btn_grp_clip_sel.clicked.connect(self.copy_selected_groups_clipboard_only)
        layout.addWidget(btn_grp_clip_sel)

    def get_options(self):
        return self.startZeroCB.isChecked(), self.formatMenu.currentText()

    # Mesh functions
    def export_all_mesh(self):
        mesh_shapes = cmds.ls(type="mesh")
        mesh_transforms = cmds.listRelatives(mesh_shapes, parent=True, fullPath=False)
        file_path = cmds.fileDialog2(fileMode=0, caption="Save All Meshes", okCaption="Save")
        if file_path:
            start_zero, fmt = self.get_options()
            export_names(mesh_transforms, file_path[0], to_clipboard=True, start_zero=start_zero, fmt=fmt)

    def export_selected_mesh(self):
        sel = cmds.ls(selection=True, dag=True, type="mesh")
        if not sel:
            QtWidgets.QMessageBox.warning(self, "Error", "No mesh selected!")
            return
        mesh_transforms = cmds.listRelatives(sel, parent=True, fullPath=False)
        file_path = cmds.fileDialog2(fileMode=0, caption="Save Selected Meshes", okCaption="Save")
        if file_path:
            start_zero, fmt = self.get_options()
            export_names(mesh_transforms, file_path[0], to_clipboard=True, start_zero=start_zero, fmt=fmt)

    def copy_clipboard_only(self):
        mesh_shapes = cmds.ls(type="mesh")
        mesh_transforms = cmds.listRelatives(mesh_shapes, parent=True, fullPath=False)
        start_zero, fmt = self.get_options()
        export_names(mesh_transforms, file_path=None, to_clipboard=True, start_zero=start_zero, fmt=fmt)

    def copy_selected_clipboard_only(self):
        sel = cmds.ls(selection=True, dag=True, type="mesh")
        if not sel:
            QtWidgets.QMessageBox.warning(self, "Error", "No mesh selected!")
            return
        mesh_transforms = cmds.listRelatives(sel, parent=True, fullPath=False)
        start_zero, fmt = self.get_options()
        export_names(mesh_transforms, file_path=None, to_clipboard=True, start_zero=start_zero, fmt=fmt)

    # Group functions
    def export_all_groups(self):
        all_transforms = cmds.ls(type="transform")
        groups = [t for t in all_transforms if not cmds.listRelatives(t, shapes=True)]
        file_path = cmds.fileDialog2(fileMode=0, caption="Save All Groups", okCaption="Save")
        if file_path:
            start_zero, fmt = self.get_options()
            export_names(groups, file_path[0], to_clipboard=True, start_zero=start_zero, fmt=fmt)

    def export_selected_groups(self):
        sel = cmds.ls(selection=True, type="transform")
        if not sel:
            QtWidgets.QMessageBox.warning(self, "Error", "No group selected!")
            return
        groups = [t for t in sel if not cmds.listRelatives(t, shapes=True)]
        if not groups:
            QtWidgets.QMessageBox.warning(self, "Error", "No valid group selected!")
            return
        file_path = cmds.fileDialog2(fileMode=0, caption="Save Selected Groups", okCaption="Save")
        if file_path:
            start_zero, fmt = self.get_options()
            export_names(groups, file_path[0], to_clipboard=True, start_zero=start_zero, fmt=fmt)

    def copy_groups_clipboard_only(self):
        all_transforms = cmds.ls(type="transform")
        groups = [t for t in all_transforms if not cmds.listRelatives(t, shapes=True)]
        start_zero, fmt = self.get_options()
        export_names(groups, file_path=None, to_clipboard=True, start_zero=start_zero, fmt=fmt)

    def copy_selected_groups_clipboard_only(self):
        sel = cmds.ls(selection=True, type="transform")
        if not sel:
            QtWidgets.QMessageBox.warning(self, "Error", "No group selected!")
            return
        groups = [t for t in sel if not cmds.listRelatives(t, shapes=True)]
        if not groups:
            QtWidgets.QMessageBox.warning(self, "Error", "No valid group selected!")
            return
        start_zero, fmt = self.get_options()
        export_names(groups, file_path=None, to_clipboard=True, start_zero=start_zero, fmt=fmt)

# Show UI
def show_ui():
    global meshExportWin
    try:
        meshExportWin.close()
        meshExportWin.deleteLater()
    except:
        pass
    meshExportWin = MeshExportUI()
    meshExportWin.show()

show_ui()
```
### Batch Rename UE v2.0 (Use in Maya) - New Method
Newer Version, Auto copy into new names. Try it below!

```ruby
import maya.cmds as cmds
from PySide2 import QtWidgets, QtCore, QtGui

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

class UnrealScriptGenerator(QtWidgets.QWidget):
    def __init__(self):
        super(UnrealScriptGenerator, self).__init__()
        self.setWindowTitle("Batch Rename UE v2.0")
        self.setFixedSize(300, 420)
        
        layout = QtWidgets.QVBoxLayout(self)

        # --- SETTINGS SECTION ---
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

        layout.addSpacing(20)
        line = QtWidgets.QFrame(); line.setFrameShape(QtWidgets.QFrame.HLine)
        layout.addWidget(line)
        layout.addSpacing(10)

        # --- ACTIONS SECTION ---
        layout.addWidget(QtWidgets.QLabel("<b>Export Actions:</b>"))
        
        self.btn_mesh = QtWidgets.QPushButton("COPY FULL UNREAL SCRIPT\n(Selected Meshes)")
        self.btn_mesh.setMinimumHeight(60)
        self.btn_mesh.setStyleSheet("background-color: #354a3b; font-weight: bold; color: #e0e0e0;")
        self.btn_mesh.clicked.connect(lambda: self.execute_copy("mesh"))
        layout.addWidget(self.btn_mesh)

        layout.addSpacing(10)

        self.btn_grp = QtWidgets.QPushButton("COPY FULL UNREAL SCRIPT\n(Selected Groups)")
        self.btn_grp.setMinimumHeight(60)
        self.btn_grp.setStyleSheet("font-weight: bold;")
        self.btn_grp.clicked.connect(lambda: self.execute_copy("group"))
        layout.addWidget(self.btn_grp)
        
        layout.addStretch()
        
        footer = QtWidgets.QLabel("Batch Rename UE v2.0")
        footer.setStyleSheet("font-size: 9px; color: #888;")
        footer.setAlignment(QtCore.Qt.AlignCenter)
        layout.addWidget(footer)

    def execute_copy(self, mode):
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

        clipboard = QtWidgets.QApplication.clipboard()
        clipboard.setText(full_code)
        
        QtWidgets.QMessageBox.information(self, "Success", f"Script for {len(items)} items copied!")

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
<img width="1318" height="762" alt="image" src="https://github.com/user-attachments/assets/7d4ec750-efc1-436e-be8b-9f06c0ee2e87" />


