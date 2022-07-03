// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © RicardoSantos

//@version=5
indicator(title='Rolling largest Impulse Murreys Lines', overlay=true, max_bars_back=500)

import RicardoSantos/FunctionArrayMaxSubKadanesAlgorithm/1 as msub
import RicardoSantos/ColorExtension/4 as colore

// helpers:
line_style_switch (string _style) =>
    switch (_style)
        ('dashed') => line.style_dashed
        ('dotted') => line.style_dotted
        ('solid') => line.style_solid
line_extend_switch (string _extend) =>
    switch (_extend)
        ('left') => extend.left
        ('right') => extend.right
        ('both') => extend.both
        ('none') => extend.none
label_textsize(string _size) =>
    switch (_size)
        ('auto') => size.auto
        ('huge') => size.huge
        ('large') => size.large
        ('normal') => size.normal
        ('small') => size.small
        ('tiny') => size.tiny
// inputs:
g_param = 'Parameters'
int window_size = input.int(defval=250, title='Window Size', minval=1, maxval=500, tooltip='size of data to search for largest impulse swing.', group=g_param)
bool direction = 'Bull' == input.string(defval='Bull', title='Direction of impulse', options=['Bull', 'Bear'], tooltip='prefered direction of impulse.', group=g_param)
float i_inversion_rate = input.float(defval=1.0, title='Inversion Rate', minval=0.0, maxval=1.0, tooltip='Inversion rate used to mirror the base line.', group=g_param)

g_style = 'Line Options'
i_base_color = input.color(defval=color.silver, title='Base', inline='b', group=g_style)
i_base_extend = line_extend_switch(input.string(defval='none', options=['left', 'right', 'both', 'none'], title='', inline='b', group=g_style))
i_base_style = line_style_switch(input.string(defval='solid', options=['dashed', 'dotted', 'solid'], title='', inline='b', group=g_style))
i_base_width = input.int(defval=1, minval=0, title='', tooltip='Base line properties.', inline='b', group=g_style)
i_invr_color = input.color(defval=color.silver, title='Inverse', inline='i', group=g_style)
i_invr_extend = line_extend_switch(input.string(defval='none', options=['left', 'right', 'both', 'none'], title='', inline='i', group=g_style))
i_invr_style = line_style_switch(input.string(defval='dashed', options=['dashed', 'dotted', 'solid'], title='', inline='i', group=g_style))
i_invr_width = input.int(defval=1, minval=0, title='', tooltip='Inverse line properties.', inline='i', group=g_style)
i_extend = line_extend_switch(input.string(defval='left', options=['left', 'right', 'both', 'none'], title='MML Extend', inline='00', group=g_style))
i_labels = input.bool(defval=true, title='', inline='00', group=g_style)
i_labels_textsize = input.string(defval='small', options=['auto', 'huge', 'large', 'normal', 'small', 'tiny'], title='Labels', inline='00', group=g_style)
i_mm01_color  = input.color(defval=color.gray, title='-2/8',                                                        inline='01', group=g_style)
i_mm01_style  = line_style_switch(input.string(defval='dotted', options=['dashed', 'dotted', 'solid'], title='',    inline='01', group=g_style))
i_mm01_width  = input.int(defval=1, minval=0, title='', tooltip='-2/8 line properties.',                            inline='01', group=g_style)
i_mm02_color  = input.color(defval=color.gray, title='-1/8',                                                        inline='02', group=g_style)
i_mm02_style  = line_style_switch(input.string(defval='dashed', options=['dashed', 'dotted', 'solid'], title='',    inline='02', group=g_style))
i_mm02_width  = input.int(defval=1, minval=0, title='', tooltip='-1/8 line properties.',                            inline='02', group=g_style)
i_mm03_color  = input.color(defval=color.aqua, title=' 0/8',                                                        inline='03', group=g_style)
i_mm03_style  = line_style_switch(input.string(defval='solid' , options=['dashed', 'dotted', 'solid'], title='',    inline='03', group=g_style))
i_mm03_width  = input.int(defval=2, minval=0, title='', tooltip=' 0/8 line properties.',                            inline='03', group=g_style)
i_mm04_color  = input.color(defval=color.gray, title=' 1/8',                                                        inline='04', group=g_style)
i_mm04_style  = line_style_switch(input.string(defval='dashed' , options=['dashed', 'dotted', 'solid'], title='',   inline='04', group=g_style))
i_mm04_width  = input.int(defval=1, minval=0, title='', tooltip=' 1/8 line properties.',                            inline='04', group=g_style)
i_mm05_color  = input.color(defval=color.gray, title=' 2/8',                                                        inline='05', group=g_style)
i_mm05_style  = line_style_switch(input.string(defval='solid' , options=['dashed', 'dotted', 'solid'], title='',    inline='05', group=g_style))
i_mm05_width  = input.int(defval=1, minval=0, title='', tooltip=' 2/8 line properties.',                            inline='05', group=g_style)
i_mm06_color  = input.color(defval=color.gray, title=' 3/8',                                                        inline='06', group=g_style)
i_mm06_style  = line_style_switch(input.string(defval='dashed' , options=['dashed', 'dotted', 'solid'], title='',   inline='06', group=g_style))
i_mm06_width  = input.int(defval=1, minval=0, title='', tooltip=' 3/8 line properties.',                            inline='06', group=g_style)
i_mm07_color  = input.color(defval=color.lime, title=' 4/8',                                                        inline='07', group=g_style)
i_mm07_style  = line_style_switch(input.string(defval='solid' , options=['dashed', 'dotted', 'solid'], title='',    inline='07', group=g_style))
i_mm07_width  = input.int(defval=2, minval=0, title='', tooltip=' 4/8 line properties.',                            inline='07', group=g_style)
i_mm08_color  = input.color(defval=color.gray, title=' 5/8',                                                        inline='08', group=g_style)
i_mm08_style  = line_style_switch(input.string(defval='dashed' , options=['dashed', 'dotted', 'solid'], title='',   inline='08', group=g_style))
i_mm08_width  = input.int(defval=1, minval=0, title='', tooltip=' 5/8 line properties.',                            inline='08', group=g_style)
i_mm09_color  = input.color(defval=color.gray, title=' 6/8',                                                        inline='09', group=g_style)
i_mm09_style  = line_style_switch(input.string(defval='solid' , options=['dashed', 'dotted', 'solid'], title='',    inline='09', group=g_style))
i_mm09_width  = input.int(defval=1, minval=0, title='', tooltip=' 6/8 line properties.',                            inline='09', group=g_style)
i_mm10_color  = input.color(defval=color.gray, title=' 7/8',                                                        inline='10', group=g_style)
i_mm10_style  = line_style_switch(input.string(defval='dashed' , options=['dashed', 'dotted', 'solid'], title='',   inline='10', group=g_style))
i_mm10_width  = input.int(defval=1, minval=0, title='', tooltip=' 7/8 line properties.',                            inline='10', group=g_style)
i_mm11_color  = input.color(defval=color.aqua, title=' 8/8',                                                        inline='11', group=g_style)
i_mm11_style  = line_style_switch(input.string(defval='solid' , options=['dashed', 'dotted', 'solid'], title='',    inline='11', group=g_style))
i_mm11_width  = input.int(defval=2, minval=0, title='', tooltip=' 8/8 line properties.',                            inline='11', group=g_style)
i_mm12_color  = input.color(defval=color.gray, title=' 9/8',                                                        inline='12', group=g_style)
i_mm12_style  = line_style_switch(input.string(defval='dashed' , options=['dashed', 'dotted', 'solid'], title='',   inline='12', group=g_style))
i_mm12_width  = input.int(defval=1, minval=0, title='', tooltip=' 9/8 line properties.',                            inline='12', group=g_style)
i_mm13_color  = input.color(defval=color.gray, title='10/8',                                                        inline='13', group=g_style)
i_mm13_style  = line_style_switch(input.string(defval='dotted' , options=['dashed', 'dotted', 'solid'], title='',   inline='13', group=g_style))
i_mm13_width  = input.int(defval=1, minval=0, title='', tooltip='10/8 line properties.',                            inline='13', group=g_style)
// prepare data:
float hl = (direction ? +1.0 : -1.0) * ta.change(high * low)

var float[] samples = array.new_float(size=window_size, initial_value=0.0)
array.unshift(samples, hl)
array.pop(samples)

// find the largest impulse in the data samples:
[ki_max, ki_start, ki_end] = msub.indices(samples)

bool is_new_impulse = ta.change(ki_start + ki_end) != 0

// initialize the lines:
var line[] lines = array.new_line(15)
var label[] labels = array.new_label(15)
if barstate.isfirst
    // base line:
    array.set(lines,  0, line.new(bar_index, 0.0, bar_index, 0.0, xloc.bar_index, i_base_extend, i_base_color, i_base_style, i_base_width))
    array.set(lines,  1, line.new(bar_index, 0.0, bar_index, 0.0, xloc.bar_index, i_invr_extend, i_invr_color, i_invr_style, i_invr_width))
    //from lowest octave to highest:
    array.set(lines,  2, line.new(bar_index, 0.0, bar_index, 0.0, xloc.bar_index, i_extend, i_mm01_color, i_mm01_style, i_mm01_width))
    array.set(lines,  3, line.new(bar_index, 0.0, bar_index, 0.0, xloc.bar_index, i_extend, i_mm02_color, i_mm02_style, i_mm02_width))
    array.set(lines,  4, line.new(bar_index, 0.0, bar_index, 0.0, xloc.bar_index, i_extend, i_mm03_color, i_mm03_style, i_mm03_width))
    array.set(lines,  5, line.new(bar_index, 0.0, bar_index, 0.0, xloc.bar_index, i_extend, i_mm04_color, i_mm04_style, i_mm04_width))
    array.set(lines,  6, line.new(bar_index, 0.0, bar_index, 0.0, xloc.bar_index, i_extend, i_mm05_color, i_mm05_style, i_mm05_width))
    array.set(lines,  7, line.new(bar_index, 0.0, bar_index, 0.0, xloc.bar_index, i_extend, i_mm06_color, i_mm06_style, i_mm06_width))
    array.set(lines,  8, line.new(bar_index, 0.0, bar_index, 0.0, xloc.bar_index, i_extend, i_mm07_color, i_mm07_style, i_mm07_width))
    array.set(lines,  9, line.new(bar_index, 0.0, bar_index, 0.0, xloc.bar_index, i_extend, i_mm08_color, i_mm08_style, i_mm08_width))
    array.set(lines, 10, line.new(bar_index, 0.0, bar_index, 0.0, xloc.bar_index, i_extend, i_mm09_color, i_mm09_style, i_mm09_width))
    array.set(lines, 11, line.new(bar_index, 0.0, bar_index, 0.0, xloc.bar_index, i_extend, i_mm10_color, i_mm10_style, i_mm10_width))
    array.set(lines, 12, line.new(bar_index, 0.0, bar_index, 0.0, xloc.bar_index, i_extend, i_mm11_color, i_mm11_style, i_mm11_width))
    array.set(lines, 13, line.new(bar_index, 0.0, bar_index, 0.0, xloc.bar_index, i_extend, i_mm12_color, i_mm12_style, i_mm12_width))
    array.set(lines, 14, line.new(bar_index, 0.0, bar_index, 0.0, xloc.bar_index, i_extend, i_mm13_color, i_mm13_style, i_mm13_width))
    //labels
    if i_labels
        array.set(labels,  2, label.new(bar_index, 0.0, '-2/8', color=i_mm01_color, style=label.style_label_right, textcolor=colore.invert(i_mm01_color), size=i_labels_textsize))
        array.set(labels,  3, label.new(bar_index, 0.0, '-1/8', color=i_mm02_color, style=label.style_label_right, textcolor=colore.invert(i_mm02_color), size=i_labels_textsize))
        array.set(labels,  4, label.new(bar_index, 0.0, ' 0/8', color=i_mm03_color, style=label.style_label_right, textcolor=colore.invert(i_mm03_color), size=i_labels_textsize))
        array.set(labels,  5, label.new(bar_index, 0.0, ' 1/8', color=i_mm04_color, style=label.style_label_right, textcolor=colore.invert(i_mm04_color), size=i_labels_textsize))
        array.set(labels,  6, label.new(bar_index, 0.0, ' 2/8', color=i_mm05_color, style=label.style_label_right, textcolor=colore.invert(i_mm05_color), size=i_labels_textsize))
        array.set(labels,  7, label.new(bar_index, 0.0, ' 3/8', color=i_mm06_color, style=label.style_label_right, textcolor=colore.invert(i_mm06_color), size=i_labels_textsize))
        array.set(labels,  8, label.new(bar_index, 0.0, ' 4/8', color=i_mm07_color, style=label.style_label_right, textcolor=colore.invert(i_mm07_color), size=i_labels_textsize))
        array.set(labels,  9, label.new(bar_index, 0.0, ' 5/8', color=i_mm08_color, style=label.style_label_right, textcolor=colore.invert(i_mm08_color), size=i_labels_textsize))
        array.set(labels, 10, label.new(bar_index, 0.0, ' 6/8', color=i_mm09_color, style=label.style_label_right, textcolor=colore.invert(i_mm09_color), size=i_labels_textsize))
        array.set(labels, 11, label.new(bar_index, 0.0, ' 7/8', color=i_mm10_color, style=label.style_label_right, textcolor=colore.invert(i_mm10_color), size=i_labels_textsize))
        array.set(labels, 12, label.new(bar_index, 0.0, ' 8/8', color=i_mm11_color, style=label.style_label_right, textcolor=colore.invert(i_mm11_color), size=i_labels_textsize))
        array.set(labels, 13, label.new(bar_index, 0.0, ' 9/8', color=i_mm12_color, style=label.style_label_right, textcolor=colore.invert(i_mm12_color), size=i_labels_textsize))
        array.set(labels, 14, label.new(bar_index, 0.0, '10/8', color=i_mm13_color, style=label.style_label_right, textcolor=colore.invert(i_mm13_color), size=i_labels_textsize))

update_label (int _n, int _x, float _y) => //{
    label _label = array.get(labels, _n)
    label.set_xy(_label, _x, _y)
//}
update_bline (int _n, int _x1, float _y1, int _x2, float _y2) => //{
    line _line = array.get(lines, _n)
    line.set_xy1(_line, _x1, _y1)
    line.set_xy2(_line, _x2, _y2)
//}
update_mline (int _n, int _x1, int _x2, float _y) => //{
    line _line = array.get(lines, _n)
    line.set_xy1(_line, _x1, _y)
    line.set_xy2(_line, _x2, _y)
    if i_labels
        update_label(_n, _x2, _y)
//}

if is_new_impulse
    float _start_value = hl2[ki_start]
    float _end_value = hl2[ki_end]
    int _x1 = bar_index - ki_start
    int _x2 = bar_index - ki_end
    float _y1 = low[ki_start]
    float _y2 = high[ki_end]
    if _end_value < _start_value
        _y1 := high[ki_start]
        _y2 := low[ki_end]

    float _range = _y2 - _y1

    update_bline(00, _x1, _y1, _x2, _y2)
    update_bline(01, _x1, _y1, _x1+(_x1-_x2), _y1+(_y2-_y1)*i_inversion_rate)
    update_mline(02, _x1, _x2, _y1 + _range * -0.250)
    update_mline(03, _x1, _x2, _y1 + _range * -0.125)
    update_mline(04, _x1, _x2, _y1)
    update_mline(05, _x1, _x2, _y1 + _range * 0.125)
    update_mline(06, _x1, _x2, _y1 + _range * 0.250)
    update_mline(07, _x1, _x2, _y1 + _range * 0.375)
    update_mline(08, _x1, _x2, _y1 + _range * 0.500)
    update_mline(09, _x1, _x2, _y1 + _range * 0.625)
    update_mline(10, _x1, _x2, _y1 + _range * 0.750)
    update_mline(11, _x1, _x2, _y1 + _range * 0.875)
    update_mline(12, _x1, _x2, _y1 + _range * 1.000)
    update_mline(13, _x1, _x2, _y1 + _range * 1.125)
    update_mline(14, _x1, _x2, _y1 + _range * 1.250)


