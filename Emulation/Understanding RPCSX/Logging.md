  case LogLevel::Always:
    color = "\e[36;1m";
  case LogLevel::Fatal:
    color = "\e[35;1m";
  case LogLevel::Error:
    color = "\e[0;31m";
  case LogLevel::Todo:
    color = "\e[1;33m";
  case LogLevel::Success:
    color = "\e[1;32m";
  case LogLevel::Warning:
    color = "\e[0;33m";
  case LogLevel::Notice:
    color = "\e[0;36m";
  case LogLevel::Trace:
    color = "";

Seems that the way that the system checks if the dir exists is to try open it and then creates the dir if it fails