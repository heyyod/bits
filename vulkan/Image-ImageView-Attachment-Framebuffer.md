**Image**
A 'description of some memory dedicated for textures'

**ImageView**
A reference to an Image which allows specifying subsections of the Image. EG specific mip map & array layers, this way you don't need to recreate the Image to render to specific layers/mipmaps. 

**Attachment**
description of the input/outputs of a renderpass. Defines the layout/format, not any actual data 

**Framebuffer**
A collection of ImageViews that correspond to the Attachments in a renderpass. EG it 'binds' imageViews together for a renderpass to use. Note that ImagelessFramebuffer allows delaying the actual binding until render-pass begin (inside command buffer recording)

