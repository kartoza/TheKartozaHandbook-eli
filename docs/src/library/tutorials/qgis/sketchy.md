
# Sketchy tree sytling on QGIS

This article focuses on the symbology of a point layer. The goal is to make the tree point layer look like it has been sketched. This is done using Geometry Generator, SVGs and wave_randomised or triangular_wave_randomised. Only QGIS 3.28.0 or other recent versions can be used for these styles.

Topics Covered:

- Symbol Layers
- Expressions
- Embedded SVGs
- Gradients
- Draw Effects
- Geometry generators


**Output:**



![sketchy_tree](https://user-images.githubusercontent.com/107027990/236208647-cb5b18ef-a2f8-440d-bb23-bf21e510a7bd.png)

**Steps:**

1. Create a point layer with a ‘crown_radius_m’ column in QGIS, using EPSG: 4326.
New Geopackage &rarr; Create a new database or select an existing one &rarr; select Point for Geometry Type &rarr; specify EPSG &rarr; Add new field ‘crown_radius_m’ &rarr; OK.

![image15](https://user-images.githubusercontent.com/107027990/236209671-1696c380-eaff-4ccf-95af-01389ba0178c.png)

2. Add a few random points and populate the columns.  

![radom_point](https://user-images.githubusercontent.com/107027990/236209819-c693ac8a-72a1-4ef0-941f-93f69e8f417b.png)

3. Under ‘Layer Styling’, select Single Symbol.
Click on ‘Simple Marker’ and Select ‘Geometry Generator’ for ‘Symbol layer type’ and ‘Polygon/MultiPolygon’ for ‘Geometry type’.

![simple_marker](https://user-images.githubusercontent.com/107027990/236210006-7d6d419c-13be-4152-8b65-eed2da90162f.png)

4. To create a buffer that depends on the crown radius value, input the following in the expression:
buffer($geometry, “crown_radius_m”) 🡪 Quotation marks (“”) must be used to refer to columns.

![buffer](https://user-images.githubusercontent.com/107027990/236210186-be487bac-e8e0-45b3-bc31-5f8be4451ab0.png)

Buffer Tool explained:

![buffer_explained](https://user-images.githubusercontent.com/107027990/236210304-25d28086-bb07-4619-9398-da3ed5d9f81c.png)

Parameters with square brackets are optional parameters.
However, the user must input geometry and distance for the buffer (in this case, it is given by the crown radius value so the distance for the buffer differs per feature). If you are going to input values for segments, cap, join and miter_limit, you can just input values while emitting the titles but they must then be in the same order as shown in the documentation. However, if the order is not applied rigidly or only some parameters need to be changed, provide the names of the parameters, for example:
buffer($geometry, “crown_radius_m”, cap:=’square’)

Output:

![buffer_output](https://user-images.githubusercontent.com/107027990/236210402-02165eaa-418a-47b4-8e29-7e71133d07fe.png)

The output of changing the segments (Note the syntax: ‘segments:=1’):

Segments = 1

![segments_1](https://user-images.githubusercontent.com/107027990/236212729-dd05ca3a-212e-4def-8171-a1a382a47cdb.png)

Segments = 2

![segments_2](https://user-images.githubusercontent.com/107027990/236212760-7c503acd-8426-4f25-93f4-a7155c53efe1.png)

Segments:= 8

![segment_8](https://user-images.githubusercontent.com/107027990/236212689-aeb9a51c-af7f-4d9f-8602-c75e1296e5c3.png)

Therefore, increasting the segments increases the smoothness of the buffer.  

5. Creating the wave effect around the points

Method 1: Using triangular_wave_randomized
Experimenting with the parameters of the tool.

![image1](https://user-images.githubusercontent.com/107027990/236213073-7fcf1709-99c9-4375-9004-157e8c723d55.png)

For the geometry parameter, we input the buffer function since the triangulated waves are created on the buffer created on a point:

triangular_wave_randomized( buffer($geometry, "crown_radius_m"), 1,1,1,1,1)

Inputting the value 1 for all the compulsory parameters is expected to create triangles around the buffer of equal shape and length (besides the starting and ending point) because that means only wavelength of 1 (since the randomly generated number is between minimum of 1 and maximum of 1, which is 1, and amplitude of 1 is selected. Therefore, this is like using triangulated_wave(buffer($geometry, “crown_radius_m”), 1, 1) instead, since this does not create the triangulated waves based on selecting a value at random from an interval.

Output:

![image29](https://user-images.githubusercontent.com/107027990/236213176-804279df-3069-44dc-86d6-a09caecac53f.png)

Increasing the range between the minimum and maximum wavelengths increases the chances of having wider angles between your triangles then increasing the range of amplitudes increases how deeply the triangles can penetrate into the buffer:

 Wavelength of 5           |  Amplitude of 5
:-------------------------:|:-------------------------:
![image9](https://user-images.githubusercontent.com/107027990/236215685-ccc9027e-c038-4b87-8d74-6d235eebe963.png)  | ![image18](https://user-images.githubusercontent.com/107027990/236215738-ea140c22-0e4a-4754-89bb-1360a782d4d8.png)

After trying out different values for wavelengths and amplitude, the following implementation is done:
 triangular_wave_randomized( buffer($geometry, "crown_radius_m"), 2, 5, 0.1, 0.5)

Output:

![out_put_smoth](https://user-images.githubusercontent.com/107027990/236213535-9956da31-87db-4008-8f9b-9a1a1d1c0622.png)

For the final touch, the ‘smooth’ tool will be used. The triangular ends of the polygon are too defined and not irregular enough to suit the ‘sketchy’ design that is intended. The iterations will be based on the id of the tree features.

![function smooth](https://user-images.githubusercontent.com/107027990/236216245-2335ac38-c05a-4b43-bd9f-3896bc1999c7.png)

smooth (triangular_wave_randomized( buffer($geometry, "crown_radius_m"),2, 5,0.1, 0.15,20), @id)

![t_wave_ran](https://user-images.githubusercontent.com/107027990/236217270-0a9def5f-5b1e-4c0e-a6f6-dadd342f0d96.png)

Method 2: Using wave_randomized.

1. We can then apply the wave randomised function to the buffer. It takes in the 6 arguments that are outlined in the image below. We start with all arguments excluding the geometry equal to 1 (initial conditions) and investigate the effect of each argument on the output by keeping all others at 1.

![image25](https://user-images.githubusercontent.com/107027990/236217449-339df347-47c9-4712-b5cb-400bb406ef36.png)

Initial output:

![image16](https://user-images.githubusercontent.com/107027990/236217762-65703064-8eba-469c-9153-f88c3fb4f708.png)

First, we look at the minimum wavelength of waves which ranges from 0 to 1. The lowest minimum wavelength yields more “petals” to our flower looking shape. This is because the wavelengths are shortened due to a lower possible minimum. This then fits more waves as the minimum wavelength number approaches zero.

Min=0.1

![min1](https://user-images.githubusercontent.com/107027990/236217889-ba51de7b-60ff-4781-9b9a-7c986f930ea1.png)

Min=0.5

![image7](https://user-images.githubusercontent.com/107027990/236218096-86daf37b-ceb9-4f64-b5d0-9434caa71aac.png)

Now, we look at the maximum wavelength of waves which ranges from the minimum wavelength to positive infinity.
Larger maximum wavelengths create more irregular looking shapes due to the greater variation in the wavelengths and thus shapes of the waves created around the buffer boundary of the point. This does not quite apply to the two below where min. wavelength = max. wavelength.

Min = 0.1, Max = 0.1

![min_max_1](https://user-images.githubusercontent.com/107027990/236218241-1e191770-076f-4df6-8056-950f964b06c7.png)

Min = 0.5, Max = 0.5

![min_max_5](https://user-images.githubusercontent.com/107027990/236218352-203aa0b4-f8eb-4ffd-a714-aa3193997430.png)

Min = 0.1, Max = 4

![min_max_4](https://user-images.githubusercontent.com/107027990/236218877-d3bf6e48-b724-490f-ae70-953d92ac6ce3.png)

Min = 0.1, Max = 8

![min_max_8](https://user-images.githubusercontent.com/107027990/236219055-9a2ebbc8-b883-424e-914e-e82879c6a680.png)

Now, we investigate the minimum and maximum amplitudes of the random waves created.  

With minimum and maximum wavelengths kept at 1,  
Minimum amplitude = 0.1, Maximum amplitude = 1:

![min_max_amp_1](https://user-images.githubusercontent.com/107027990/236219255-a3d7480b-6354-4cbf-8397-ee927e945885.png)

We immediately observe that the wave amplitude affects the distance that the “petals” on the circumference can approach the centre of the point (the centroid).

Minimum amplitude = 0.1, Maximum amplitude = 0.3:

![min_max_amp_3](https://user-images.githubusercontent.com/107027990/236219364-b35ea07c-27b2-4635-93d7-f17bf2a4bcec.png)

And now with the smaller minimum and maximum amplitudes, we get closer to the desired sketchy shape.  

1. We can now duplicate the Marker
![image8](https://user-images.githubusercontent.com/107027990/236219515-5f89442e-138d-4c17-bfda-e67172e4d05a.png)

1. Make the first Marker’s Geometry Generator fill colour Transparent.
![image14](https://user-images.githubusercontent.com/107027990/236219605-719e5d8c-c0dc-4d8d-b7ae-158b933fd9fd.png)

Then click on the green plus sign to Add Symbol Layer. Once added, change the Symbol Layer Type to ‘Outline: Simple Line’ and match the Stroke Widths. Playing around with the stroke widths yields different sketchy outputs.

![add_symbol](https://user-images.githubusercontent.com/107027990/236219734-ec9c9442-4a82-485c-ae88-eede4f674d29.png)

You can also continue to duplicate the Geometry Generator layers with different seed values to create more sketch lines.

![dup](https://user-images.githubusercontent.com/107027990/236219838-c61386d5-de15-4784-804d-32d926cebe2b.png)

8. Once the outlines look sketchy, we can change the Symbol Layer Type to Gradient Fill.
This shows a simple Gradient Fill from the top to the bottom of the symbol.

![symbol_layer_type](https://user-images.githubusercontent.com/107027990/236220144-46297f0e-0bef-4e1e-adc6-ea9bda11cca2.png)

The reference point 1 tells us where the light of the gradient should originate. The box below shows the top-left represents (x,y) = (0,0) and the bottom-right is (1,1). As such your gradient direction is determined by the (x,y) values in the first and second reference points.

![xy](https://user-images.githubusercontent.com/107027990/236221514-d38111bb-e5c9-46c0-8727-c4b67d2ceae7.png)

Creating the centroid marker.

![centroid_marker](https://user-images.githubusercontent.com/107027990/236221688-baddb70b-ae3e-49b5-9a57-d2a800c04d58.png)

This can be done on Inkscape. Once you have created your own svg in Inkscape, save it.  You can then use your Marker in QGIS under ‘Symbology’ of the third layer of your style, select ‘SVG marker’ and then choose the marker you created in Inkscape This embeds the svg on top of the tree layer.

![image_17](https://user-images.githubusercontent.com/107027990/236222173-eeff5973-7e35-4b57-b87d-31968c656e37.png)

Adding Leafy Texture.

![image_21](https://user-images.githubusercontent.com/107027990/236222460-f6e889ff-6e19-41a7-9a18-3a43ab323f90.png)

NB: Trees have leaves, so they must be given a leafy texture

![image_19](https://user-images.githubusercontent.com/107027990/236222719-441aa40c-a436-40dd-b7dc-1e065a755a1b.png)

Finally the sketchy trees will have a leafy texture with small ‘svg’ markers on top:

![image_18](https://user-images.githubusercontent.com/107027990/236222869-0e8a5171-f1fa-4e5d-80a6-be02370a3ea8.png)

10. Radius Feature
Now we will add the line going from the centre of the point to the circumference of the feature.
1. Duplicate a feature layer and move to the top then change the Geometry Type to ‘LineString/MultiString’.

![image_lend](https://user-images.githubusercontent.com/107027990/236223036-988addbc-e111-4f02-980d-05090228ff41.png)

2. To generate the squiggly line, we will use two new functions: ‘with_variable’ and ‘make_line’.

![function_make_line](https://user-images.githubusercontent.com/107027990/236223348-35b3cfd8-2f40-445a-9e8d-234e5e5e9576.png)

In the expression field, input:
 with_variable('poly',buffer($geometry, "crown_radius_m"),    smooth(triangular_wave_randomized( make_line( centroid(@poly), start_point(@poly)), 2, 5, 0.1, 0.15, 20), 10)).

 3. Duplicate the layer created in step 2, and input:
with_variable('poly',buffer($geometry, "crown_radius_m"), smooth(triangular_wave_randomized( make_line( centroid(@poly), start_point(@poly)),  2, 4, 0.1, 0.15, 20), 10))

 4. Now that you have sketched a squiggly line from radius to circumference, we need to add an arrow on each end. This is done by duplicating the layer from Steps 2 or 3 and changing the Symbol Layer Type to ‘Marker Line’.  Under the Marker Line Properties, make sure ‘On First vertex’ is ticked and untick ‘Rotate marker’ to follow line direction. Change the size to 6 millimetres.

On the ‘Simple Marker’ layer, scroll to the bottom and select the icon below. Additionally, change the rotation property to 180 degrees,

5. Duplicate the layer created in step 4 but change the selection from ‘On First vertex’ to ‘On last vertex’ and keep the rotation on 0 degrees

![dup_layer](https://user-images.githubusercontent.com/107027990/236223698-7536f198-b7aa-413d-b289-887d2a5b361a.png)

Label feature
Now that we have a squiggly line with two arrows and a sketchy tree designed, the only thing missing is the label. Similar to the rest of the design, we also want this to look hand-written.

1. Right click on layer > Properties > Label > Single Labels > Set Value to ‘concat(“crown_radius_m”, ‘ m’). This is to show the unit for the radius without having to input ‘m’ in the data record because we need this layer to remain in decimal data type. Therefore, concat combines the numeric value of the radius and adds the suffix ‘m’ to it.

![func_concat](https://user-images.githubusercontent.com/107027990/236223986-b8475b2a-7486-4a02-a9a7-d0841de5e323.png)

2. Select Bradley Hand ITC for font.
3. Go to Placement &rarr; Change Mode to ‘Offset from Point’ and select the central position.

After all the steps, you should have a symbology that looks hand-sketched, showing a hand-sketched top-view of a tree, a hand-sketched arrow from the centre to circumference and handwritten crown measurement with a unit.
