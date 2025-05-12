require 'minitest/autorun'

class PointCart
  attr_reader :x, :y

  def initialize(x, y)
    @x = x.to_f
    @y = y.to_f
  end

  def to_s
    "[#{@x}, #{@y}]"
  end

  def to_pol
    r = Math.sqrt(@x**2 + @y**2)
    theta = Math.atan2(@y, @x)
    PointPol.new(r, Angle.new(theta * 180 / Math::PI))
  end

  def -(other)
    PointCart.new(@x - other.x, @y - other.y)
  end

  def +(other)
    PointCart.new(@x + other.x, @y + other.y)
  end

  def |(other)
    Math.sqrt((@x - other.x)**2 + (@y - other.y)**2)
  end
end

class Angle
  attr_reader :degrees

  def initialize(degrees)
    @degrees = degrees % 360
    @degrees += 360 if @degrees < 0
    @degrees %= 360
  end

  def to_s
    "#{@degrees}°"
  end

  def to_radian
    rad = @degrees * Math::PI / 180
    rad.round(2)
  end
end

class PointPol
  attr_reader :r, :theta

  def initialize(r, theta)
    @r = r.to_f
    @theta = theta.is_a?(Angle) ? theta : Angle.new(theta)
  end

  def to_s
    "[#{@r}, #{@theta.to_s}]"
  end

  def to_cart
    x = @r * Math.cos(@theta.to_radian)
    y = @r * Math.sin(@theta.to_radian)
    PointCart.new(x, y)
  end
end

class Line
  attr_reader :p1, :p2, :translation

  def initialize(point1, point2)
    c1 = point1.is_a?(PointCart) ? point1 : point1.to_cart
    c2 = point2.is_a?(PointCart) ? point2 : point2.to_cart
    if c1.x**2 + c1.y**2 <= c2.x**2 + c2.y**2
      @p1, @p2 = c1, c2
    else
      @p1, @p2 = c2, c1
    end
    @translation = nil
  end

  def to_s
    "#{@p1.to_s}---#{@p2.to_s}"
  end

  def rotate_2d(angle)
    a = angle.is_a?(Angle) ? angle : Angle.new(angle)
    rotate_point = lambda do |pt|
      r = Math.sqrt(pt.x**2 + pt.y**2)
      theta = Math.atan2(pt.y, pt.x)
      new_theta = theta + a.to_radian
      x_new = r * Math.cos(new_theta)
      y_new = r * Math.sin(new_theta)
      PointCart.new(x_new, y_new)
    end
    p1_rot = rotate_point.call(@p1)
    p2_rot = rotate_point.call(@p2)
    Line.new(p1_rot, p2_rot)
  end
end

class MainTest < Minitest::Test
  def test_line_initialization
    p1 = PointCart.new(0, 1)
    p2 = PointCart.new(0, 4)
    line = Line.new(p1, p2)
    assert_equal p1.x, line.p1.x
    assert_equal p1.y, line.p1.y
    assert_equal p2.x, line.p2.x
    assert_equal p2.y, line.p2.y
  end

  def test_line_to_s
    p1 = PointCart.new(0, 1)
    p2 = PointCart.new(0, 4)
    line = Line.new(p1, p2)
    assert_equal "[0.0, 1.0]---[0.0, 4.0]", line.to_s
  end

  def test_line_rotate_2d
    p1 = PointCart.new(0, 1)
    p2 = PointCart.new(0, 4)
    line = Line.new(p1, p2)
    angle = Angle.new(90)
    rotated = line.rotate_2d(angle)
    assert_in_delta(-1.0, rotated.p1.x, 0.01)
    assert_in_delta(0.0, rotated.p1.y, 0.01)
    assert_in_delta(-4.0, rotated.p2.x, 0.01)
    assert_in_delta(0.0, rotated.p2.y, 0.01)
  end
end
