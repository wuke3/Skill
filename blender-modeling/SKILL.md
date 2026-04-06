---
name: blender-modeling
description: "Create, modify, and render 3D models and scenes in Blender using natural language commands powered by Blender MCP. Use this skill whenever the user wants to: create 3D objects (cubes, spheres, cylinders, planes, etc.), modify mesh geometry, apply materials and textures, set up lighting and cameras, execute Python scripts in Blender, manage scene properties, or any task related to Blender 3D modeling. Make sure to use this skill when users mention Blender, 3D modeling, 3D objects, materials, textures, lighting, rendering, or scene creation - even if they don't explicitly mention 'MCP' or 'skill'."
---

# Blender Modeling Skill

This skill enables AI-powered 3D modeling in Blender through the Model Context Protocol (MCP). It provides comprehensive guidance for creating, modifying, and rendering 3D content using natural language instructions.

## Prerequisites

- Blender installed (version 3.0+ recommended)
- BlenderMCP addon installed and configured
- Blender MCP server running (typically on localhost:9876)

## Available Tools

The Blender MCP exposes these tool categories:

### Scene & Object Query
- `get_scene_info` - Retrieve current scene state, objects, and properties
- `get_object_info` - Get detailed information about specific objects
- `list_objects` - List all objects in the current scene

### Object Creation
- `create_object` - Create basic shapes (cube, sphere, cylinder, cone, torus, plane, UV_sphere, ico_sphere)
- `create_material` - Create new materials
- `create_light` - Add lighting sources (sun, point, spot, area)

### Object Modification
- `transform_object` - Move, rotate, scale objects
- `modify_mesh` - Edit mesh properties (vertices, edges, faces)
- `apply_material` - Assign materials to objects
- `delete_object` - Remove objects from scene

### Advanced Operations
- `execute_python` - Run custom Python code in Blender
- `set_properties` - Modify object or scene properties
- `render_scene` - Trigger rendering

## Workflow

### 1. Scene Assessment

Before making changes, always understand the current scene state:

```
Use get_scene_info to check:
- What objects already exist
- Current lighting setup
- Camera positions
- Material assignments
```

### 2. Planning and Execution

**For object creation:**
1. Clarify geometry type and dimensions
2. Determine location in 3D space
3. Plan material/color requirements
4. Execute creation commands
5. Verify object was created successfully

**For object modification:**
1. Identify target object(s) using get_object_info
2. Plan specific modifications
3. Execute changes incrementally
4. Verify changes before proceeding

### 3. Material Workflow

```
Material Creation Pattern:
1. create_material with base color
2. Adjust roughness, metallic, transmission
3. Apply to object with apply_material
4. Fine-tune if needed
```

### 4. Lighting Setup

```
Lighting Principles:
- Three-point lighting: Key light, fill light, rim light
- HDRI for realistic environment lighting
- Sun light for directional shadows
- Point lights for localized illumination
- Adjust energy/intensity as needed
```

### 5. Camera Setup

```
Camera Best Practices:
- Position camera using clear 3D coordinates
- Set appropriate focal length (50mm for realistic)
- Frame subject with proper composition
- Use empty objects as camera targets
```

## Object Creation Guidelines

### Primitive Shapes

| Shape | Best Use Case | Key Parameters |
|-------|--------------|-----------------|
| Cube | Architecture, hard surface | size, location |
| Sphere | Organic, natural forms | radius, subdivisions |
| Cylinder | Pipes, columns, rods | radius, depth, vertices |
| Cone | Tips, pointed objects | radius1, radius2, depth |
| Torus | Rings, tires, donuts | major_radius, minor_radius |
| Plane | Ground, surfaces | size |

### Dimension Specification

**Always use appropriate units:**
- Small objects: 0.1-2 meters
- Medium objects: 2-10 meters
- Large objects: 10+ meters
- Consider scale relative to scene context

### Naming Convention

```
Use descriptive names:
- "character_head" not "Object.003"
- "wooden_table" not "Mesh.001"
- "emissive_sign" not "Light"
```

## Material Guidelines

### PBR Material Properties

```
Key PBR Parameters:
- base_color: RGB or hex color
- roughness: 0.0 (mirror) to 1.0 (matte)
- metallic: 0.0 (dielectric) to 1.0 (metal)
- transmission: 0.0 (opaque) to 1.0 (glass-like)
- emission: For glowing materials
```

### Common Material Presets

| Material | base_color | roughness | metallic |
|----------|-----------|-----------|----------|
| Metal (gold) | #FFD700 | 0.2 | 1.0 |
| Metal (silver) | #C0C0C0 | 0.1 | 1.0 |
| Plastic (white) | #FFFFFF | 0.5 | 0.0 |
| Wood (oak) | #8B4513 | 0.7 | 0.0 |
| Glass | #FFFFFF | 0.0 | 0.0 |
| Rubber | #333333 | 0.9 | 0.0 |

## Transform Operations

### Coordinate System

```
Blender uses Y-forward, Z-up coordinate system
- X: Left/Right
- Y: Forward/Backward (into screen)
- Z: Up/Down
```

### Transform Order

```
When moving and rotating:
1. Apply rotation first
2. Then apply location
3. Use parent-child relationships for complex hierarchies
```

## Python Scripting

### When to Use execute_python

- Complex procedural operations
- Repetitive tasks with variations
- Custom modifiers or effects
- Advanced material setups

### Script Template

```python
import bpy

# Clear existing objects (optional)
# bpy.ops.object.select_all(action='SELECT')
# bpy.ops.object.delete()

# Create new object
bpy.ops.mesh.primitive_cube_add(size=2, location=(0, 0, 0))
obj = bpy.context.active_object
obj.name = "MyCube"

# Add material
mat = bpy.data.materials.new(name="MyMaterial")
mat.use_nodes = True
bsdf = mat.node_tree.nodes["Principled BSDF"]
bsdf.inputs["Base Color"].default_value = (0.8, 0.2, 0.2, 1.0)
obj.data.materials.append(mat)
```

### Useful Blender Python Commands

```
Object selection: bpy.context.selected_objects
Active object: bpy.context.active_object
Mode switching: bpy.ops.object.mode_set(mode='EDIT')
Apply transforms: bpy.ops.object.transform_apply(location=True, rotation=True, scale=True)
```

## Rendering Guidelines

### Render Settings

```
Engine recommendations:
- Eevee: Real-time, fast preview
- Cycles: Photorealistic, longer render time

Output settings:
- Resolution: 1920x1080 (HD), 3840x2160 (4K)
- File format: PNG (lossless) or JPEG (smaller)
```

### Pre-render Checklist

```
Before rendering:
1. Check camera position and framing
2. Verify lighting setup
3. Ensure materials are properly assigned
4. Set appropriate render engine
5. Configure output path
```

## Error Handling

### Common Issues and Solutions

| Issue | Solution |
|-------|----------|
| Object not visible | Check location (might be inside another object or off-camera) |
| Material looks wrong | Verify material is assigned and node setup is correct |
| Black render | Check if lights exist and have energy > 0 |
| Deformed mesh | Ensure scale is applied (Ctrl+A) |
| Performance issues | Reduce subdivisions, use Eevee for preview |

### Debugging Steps

```
1. get_scene_info - Verify object exists
2. get_object_info - Check transform and properties
3. Try simpler geometry first to isolate issue
4. Use execute_python with print statements
```

## Workflow Examples

### Example 1: Create a Stylized Tree

```
1. get_scene_info (assess scene)
2. create_object type="cone" name="tree_leaves" radius1=2, radius2=0, depth=4
3. create_object type="cylinder" name="tree_trunk" radius=0.3, depth=3
4. transform_object move both objects together
5. create_material green for leaves, brown for trunk
6. apply_material to respective objects
```

### Example 2: Set Up Product Showcase

```
1. get_scene_info
2. create_object type="cylinder" name="platform" radius=2, depth=0.2
3. create_material (dark gray, low roughness)
4. apply_material to platform
5. create_object type="cube" name="product" size=1
6. Position product above platform
7. create_light type="area" for soft shadows
8. Position camera at 45-degree angle
```

### Example 3: Create Glass Material

```
1. create_material name="glass"
2. Set base_color to white/light blue
3. Set roughness to 0.0
4. Set metallic to 0.0
5. Set transmission to 1.0 (for Cycles)
6. Apply to object
```

## Performance Tips

```
1. Use appropriate mesh density (don't over-subdivide)
2. Use instances for repeated objects
3. Parent objects instead of joining (preserves materials)
4. Use Eevee for real-time preview
5. Bake textures when possible
6. Use decimation for imported meshes
```

## Best Practices Summary

1. **Always assess scene first** - Know what exists before adding
2. **Use descriptive names** - Easier to manage and debug
3. **Work incrementally** - Make small changes and verify
4. **Organize with collections** - Group related objects
5. **Apply transforms regularly** - Avoid scale/rotation issues
6. **Save frequently** - Blender can crash
7. **Use appropriate units** - Keep scale consistent
8. **Leverage materials** - Good materials make models shine
9. **Plan lighting** - It defines the mood
10. **Think about render** - Design with the final output in mind
