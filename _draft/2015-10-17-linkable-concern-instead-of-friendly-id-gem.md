module Linkable
  extend ActiveSupport::Concern

  included do
    def self.linkable link_field, *candidate_field
      @@linkable_link_field = link_field
      @@linkable_candidate_field = candidate_field
      before_save :generate_link
    end
  end

  def generate_link
    candidate_text = nil
    if @@linkable_candidate_field.class == Array
      @@linkable_candidate_field.each do |candidate_field|
        if candidate_field.class == Array
          candidate_text_array = candidate_field.map {|f| self.send f}
          next if candidate_text_array.select(&:blank?).present?
          candidate_text = candidate_text_array.join('-')
        else
          candidate_text = self.send candidate_field
        end
        break if candidate_text.present?
      end
    else
      candidate_text = self.send @@linkable_candidate_field
    end
    return if candidate_text.blank?
    link = core_link = prepare_link candidate_text
    num = 1
    while self.class.where( @@linkable_link_field => link).where.not( id: self.id).exists?
      num += 1  
      link = "#{core_link}-#{num}"
    end
    self.send "#{@@linkable_link_field}=", link
  end

  def prepare_string input
    # http://stackoverflow.com/questions/20224915/iconv-will-be-deprecated-in-the-future-transliterate
    result_ascii = ActiveSupport::Multibyte::Unicode.normalize(input, :kd).chars.grep(/\p{^Mn}/).join('')

    result = result_ascii.strip.downcase.gsub(/\s+/,".")
    result
  end

  def prepare_link input
    temp = prepare_string input
    temp = temp.gsub(".","-") # dot . conflicts with rails routes request type
    temp = temp.gsub(",","-") # comma , conflicts with joining customers website maps with ,
    temp
  end
end
