# music-recognition-2
require 'openssl'
require 'base64'
require 'net/http/post/multipart'
requrl = "http://identify-eu-west-1.acrcloud.com/v1/identify"
access_key = "38fe661b266535bc3af0f98b3251843eY"
access_secret = "oU2255X8SHuiZIcFKXhQCtx9aWNl27L1RWn4GngV"
http_method = "POST"
http_uri = "/v1/identify"
data_type = "audio"
signature_version = "1"
timestamp = Time.now.utc().to_i.to_s

string_to_sign = http_method+"\n"+http_uri+"\n"+access_key+"\n"+data_type+"\n"+signature_version+"\n"+timestamp

digest = OpenSSL::Digest.new('sha1')
signature = Base64.encode64(OpenSSL::HMAC.digest(digest, access_secret, string_to_sign))

file_name = ARGV[0]
sample_bytes = File.size(file_name)

url = URI.parse(requrl)
File.open(file_name) do |file|
  req = Net::HTTP::Post::Multipart.new url.path,
    "sample" => UploadIO.new(file, "audio/mp3", file_name),
    "access_key" =>access_key,
    "data_type"=> data_type,
    "signature_version"=> signature_version,
    "signature"=>signature,
    "sample_bytes"=>sample_bytes,
    "timestamp" => timestamp
  res = Net::HTTP.start(url.host, url.port) do |http|
    http.request(req)
  end
  puts(res.body)
end
